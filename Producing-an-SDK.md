# Producing an SDK

## Overview

The Pulumi SDK is made up of the CLI and language specific packages (e.g `@pulumi/pulumi`, `@pulumi/aws`, and `@pulumi/cloud`).  The CLI and packages ship independently from one another.

A release of the SDK includes the following:
- [ ] Build and release the subset of the entire stack that is needed for the desired release
- [ ] Validate the release in `home` and `pulumi-service` repos
- [ ] List the release and update documentation
- [ ] Update examples and/or templates as needed

In the past, releasing a new SDK has meant building a new version of the CLI and `@pulumi/pulumi`, and then building "up the stack", updating each package to depend on the just released version of all its dependencies. We did this in the past because multiple copies of the base pulumi runtime package could not be loaded into an application at once, so we wanted to make we had a consistent set of packages that would all depend on a unified `@pulumi/pulumi` reference.  We no longer have this dependency, so we should be able to publish packages independently of one another.

Every build out of master or a branch named `release/...` will publish a release.  We use git tags for versioning today, so the build number is based on the tags present when we do the build. This means we end up tagging a build *before* we produce it, which is not great but that is where we are today.

## Building and releasing a package

While we have not yet set the major version of any of our packages to be one or higher, and hence can make breaks at any point in development, we try to bump the minor version number only when there has been a breaking change. That means folks using `^` version constraints will never float across minor versions and hence it should always be safe to upgrade their packages. So before making a release, understand if there have been any breaking changes from the previous release. If not, the process for doing a new release is straightforward.

We have a release branch in every repository for each minor version we've released. When you want to release a new patch release, you first want to merge the existing code in master (at whatever commit you want to release) into the corresponding release branch

```sh
# Check out the release branch locally
$ git checkout release/0.15

# Ensure our local copy has all the changes from origin
$ git pull

# Merge is changes from master
$ git merge master
```

If there are conflicts, fix them up and then commit the resulting merge.

After the merge has been committed, we need to tag the commit with the version number to use for the release. Since there are no breaking changes, we'll just bump the patch version from the previous release. If you don't know the old version, running `./scripts/get-version` from the root of the repository will tell you the current version.

Our build uses git tags to determine the current version, so you need to tag the current commit to set it's version. We use amended tags so that our tags have timestamps associated with them. This ensures if we tag the same commit multiple times, we get the "newest" version each time we compute the version from the tag. For example, if the last release of a package was `v0.15.0`, and we are ready to release `v0.15.1`, after merging, run the following:

```
$ git tag -a -m "v0.15.1" v0.15.1
```

After tagging, you can run `./scripts/get-version` to ensure the command worked as expected. Note the leading `v` on the tag, which is important.

In addition to tagging the commit we are going to release, we need to tag `master`'s HEAD commit so that future builds from master are versioned as `-dev` builds of the next release.

In this case, when we've tagged `v0.15.1` in the release branch, we want to create a tag called `v0.15.2-dev` on the main commit in the master branch:

```
$ git checkout master
$ git tag -a -m "v0.15.2-dev" v0.15.2-dev
```

Once this is complete, we can push the changes to GitHub with:

```
$ git push release/0.15 --tags
```

Which will push both the release branch as well as all the tags we've created.

There is one final step. Pushing these tags and branches to GitHub will cause multiple Travis builds to be queued.  One for each pushed branch and one for each pushed tags.  Of these builds, the only interesting one is the tagged build for the release we want to build.

To save Travis resources, you should go ahead and cancel the other builds. You can do this in the Travis UI at  `https://travis-ci.com/`. 

When you are on this page, in the left will be a list of queued and running builds. There should be at least three for the repository you just pushed. If you click on an individual build, it will bring up the build details, which will include the branch or tag that the build corresponds to. You can click the "Cancel job" button to cancel the build.

If you don't do this, we'll both do extra work in CI and one of the two jobs in the release branch will end up failing (it will try to publish a package to NPM with the same version as the other build that just completed, and NPM will reject that).

## Building and releasing the entire stack

In each repository, we have a branch called `release/0.<minor-version>` (for example, `release/0.11` for the 0.11.X versions of packages).  We create this branch when we want to do the first release of a minor version, and then use it for all future patch releases.  We have a separate branch so we can easily go back and make hot fixes if needed.

Since we use git tags to compute our versions, we tag commits in this branch with `v0.<minor-number>.<patch>` with an optional `-rcX` pre-release tag, while doing pre-releases.  So the normal flow for releasing a package is:

1. Get the code in the release branch.  We either do this by merging master into the release branch (if we are comfortable taking all the changes) or `git cherry-pick`'ing changes from master into the release branch.
2. tag HEAD of `master` with a new tag for the `-dev` version of the next release:  For example, If we merged code into the release branch to produce `v0.11.3`, we should tag HEAD of master with `v0.11.4-dev`.  This ensures that the next build out of master will have the correct version.  You'll want to push the tags with `git push origin --tags` after tagging, and you can also abort the job Travis will queue on push (we've already built the code, no need to build it again).
3. Do any necessary dependency updates.  The `Gopkg.toml` files that we have in the release branches use branch constraints on packages, so `dep ensure -update <repository-path>` should do the thing you most often want, which is to update your dependencies based on the `HEAD` of their release branches.  In the case of our provider packages generated (i.e. the ones generated using `tfgen`) there's a section in `resources.go` which must also be updated, which specifies the version of dependencies for the packages we generate (see https://github.com/pulumi/pulumi-aws/blob/0d75d81515ddda0438b44969fdcecd007cd1dec7/resources.go#L1287-L1299 as an example).  Today we always bump these versions as well, to ensure that only one copy of each package ends up in a user's `node_modules` folder.  In repositories like `pulumi-cloud` we hard-code versions into `package.json` in the `peerDependencies` section, and our publish processes promotes them to actual dependencies, so these need to be updated as well.
4. Tag the commit, creating an annotated tag.  We use annotated tags because they record time stamps, allowing `git describe --tags` to pick the "newer" one, if a single commit is tagged multiple times.  `git tag -a <tag-name> -m <message>` is the way to do this.  I usually just a message which is the same as the tag name.  As an example: `git tag -a v0.11.0-rc1 -m "v0.11.0-rc1"`.
5. Push the new commits, as well as the tags: `git push origin release/0.11 ; git push origin --tags`
6. Travis will queue a build for both the push and the tag, so I normally go in to the travis console and abort one of the jobs, to save CI resources.  Note that we can't publish the same version of a package twice, so if we don't abort one, the job that runs second will fail when it tries to publish.

We repeat the above process in dependency order for {`pulumi`, `pulumi-terraform`, `pulumi-aws`, `pulumi-azure`, `pulumi-kubernetes`, `pulumi-aws-infra`, and finally `pulumi-cloud`}.

## Listing the release and updating documentation

In order for customers to get the new release, you need to update the [install page on docs.pulumi.com](https://docs.pulumi.com/install/). Note that the /releases/ endpoint on `docs.pulumi.com` will proxy any requests for non-prerelease published SDKs to the right S3 bucket.

1. In your `pulumi/docs` repository, update [`install/index.md`](https://github.com/pulumi/docs/blob/master/install/index.md). Update the variable `installer_version` in the YAML [front matter](https://jekyllrb.com/docs/frontmatter/). Verify that the links are correct when you generate the site locally.
1. Generate a new changelog, following the instructions in the docs repo readme [Generating a change log](https://github.com/pulumi/docs#generating-a-change-log). 
   - A list of contributors will be generated, based on pull request authors. Change the text to something like, "We'd like to thank these  community members for their contributions to this release". Remove Pulumi team members from the list.
1. Update the quickstart and reference content to integrate any breaking changes. Add or remove known issues from [known-issues.md](https://github.com/pulumi/docs/blob/master/reference/known-issues.md).
1. Update the READMEs and `package.json` in [Pulumi examples](https://github.com/pulumi/examples). 
1. Generate new API reference docs, following the instructions in https://github.com/pulumi/docs#development. 
1. Edit [latest-version](https://github.com/pulumi/docs/blob/master/latest-version) to set the latest CLI version that the CLI will use for version checks. (The CLI will not request this directly, but through an API endpoint on the service that proxies requests to this file on the docs site.)
1. Install the latest `pulumi` CLI. Then, generate new CLI documentation by running `pulumi gen-markdown reference/cli` in the docs repo. 
1. Merge your changes into master. Then, create a new PR to merge into production.
1. To push the new content live, follow the instructions in [Updating Website Content](https://github.com/pulumi/home/wiki/Updating-the-Docs-Website#updating-website-content).

## Release Validation

Prior to release, we run two pieces of E2E validation:
1. Run the LM test in `pulumi/home` with the full set of new versions of CLI and packages
2. Update the service to use the latest CLI (and optionally, new packages especially if there are new patch releases without breaking changes that can be updated in the lock files).