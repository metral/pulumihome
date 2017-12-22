We have many repos and need to manage dependencies between them.

# Go

For our Go repos, we use [Dep](https://github.com/golang/dep), the soon-to-be-official tool for managing Go dependencies.

Unfortunately, it's not self-evident how best to manage dependencies between repos that are under active development.  The below guidelines articulate the (admittedly ever-evolving) process that we currently use.

## Repo Tiers

First, we split our repos up into two tiers:

* SDK repos (`pulumi/pulumi`, `pulumi/pulumi-terraform`, `pulumi/pulumi-aws`, etc)
* Service repos (`pulumi/pulumi-ppc`, `pulumi/pulumi-service`, etc)

Anywhere possible, we use a prebuilt SDK at a particular version, rather than building it from source.  This includes the service repos -- with some caveats (see below) -- and any other repos not listed here, like `pulumi/examples`, etc.

## Updating SDK Dependencies

The SDK repos depend on each other using a `[[constraint]]` that specifies the `master` branch.  This ensures that we can use `dep ensure -update` to pick up recent commits from dependency repos very easily.

This means, for example, that if you want to pick up the latest `pulumi/pulumi` from `pulumi/pulumi-aws`, you simply run

```bash
pulumi-aws/$ pulumi ensure -update github.com/pulumi/pulumi
```

This will update your `Gopkg.lock` file's entry for the `pulumi/pulumi` repo to the latest commit in `master`.  This is the standard development workflow in between releases, where you only care about getting "the latest."

As we build official releases, however, the process is a little different.  This is because we use semver tags as we perform releases.  So, assuming you've committed and tagged `pulumi/pulumi` version `v0.9.7`, you consume it as:

```bash
pulumi-aws/$ pulumi ensure -update github.com/pulumi/pulumi@v0.9.7
```

The updated lockfile must be committed and tagged accordingly so that upstream dependents can consume the result.

## Updating Service Dependencies

Deploying a new SDK to `pulumi/pulumi-ppc` or `pulumi/pulumi-service` generally entails simply updating the installation scripts to refer to the newly released SDK version.

But this is not entirely sufficient, because these repos *also* have source dependencies on the `pulumi/pulumi` repo, which is also the foundation of the SDK.  (This is to share API types and various logic.)  So, we need to use `dep` here too.

The process is not terribly dissimilar to the one explained above for the SDK's inter-dependencies.  These repos also use a `[[constraint]]` for branch `master`, so the latest is easy to consume.  For instance, to get the latest `pulumi/pulumi` into `pulumi/pulumi-ppc`:

```bash
pulumi-ppc/$ pulumi ensure -update github.com/pulumi/pulumi
```

This will pick up the latest, and the same syntax as above can be used to consume a released version:

```bash
pulumi-ppc/$ pulumi ensure -update github.com/pulumi/pulumi@v0.9.7
```

**There is one major subtlety here to be aware of**.  There are now two versions on "SDK code" here: the actual released SDK bits, used dynamically, and the `pulumi/pulumi` statically linked dependency.  These must be compatible!  Because of our approach to versioning, it is generally safe for the statically linked dependencies to "race ahead" of the SDK.  This can happen sometimes if features in lower layers need to be consumed to enable the service repos to advance ahead of the SDK.  For instance, perhaps a new API has been added, and the shared definitions need to be consumed by the service repos before the new SDK can be released and used against the service.  This is fine but must be staged accordingly.