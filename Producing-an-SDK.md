# Producing an SDK

## Overview

Our SDK is comprised of these repos, in order of dependencies:

* [`pulumi/pulumi`](https://github.com/pulumi/pulumi): the core engine
* [`pulumi/pulumi-terraform`](https://github.com/pulumi/pulumi-terraform): the Terraform bridge for our resource providers
* [`pulumi/pulumi-aws`](https://github.com/pulumi/pulumi-aws): the AWS provider package
* [`pulumi/pulumi-cloud`](https://github.com/pulumi/pulumi-cloud): the Pulumi Cloud Framework packages

In general, we build and tag each one using semvers.  The process of updating versions is [outlined here](https://github.com/pulumi/home/wiki/Managing-Repo-Versions), however suffice it to say, it is manual and labor intensive at the moment.

:construction: *We are actively working on automation to make this more...automated.*

As we tag versions, our CI jobs will publish builds (per the below).  As soon as all of the above SDK repos are built, a job in [`pulumi/sdk`](https://github.com/pulumi/sdk) will run to produce the final package.

After doing this, you will need to update the consuming services, [`pulumi/pulumi-ppc`](https://github.com/pulumi/pulumi-ppc), [`pulumi/pulumi-service`](https://github.com/pulumi/pulumi-service), and [`pulumi/home`](https://github.com/pulumi/home) to consume the new SDK. This will require updates to the version specified in the `.travis.yml` file in addition to normal edits to `Gopkg.toml` followed by `dep ensure update`.

## Publishing Builds

When Travis or AppVeyor build any branch (or tag) successfully, they run our publishing scripts to upload binaries to [s3://eng.pulumi.com/releases](https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/releases/?region=us-east-1#).

We publish each build under three different names:
1. The full SHA of the commit we are building
2. The name of branch we are building (e.g. master)
3. The value returned by `git describe --tags` which is a name based on the newest tag reachable from the commit we are building.

Each repository produces an archive that is designed to be extracted into a root folder (`PULUMI_DIR`) which is the install location of Pulumi on a end user machine.

To create an SDK, we need to combine builds from all the branches we care about and then archive the combined build. Because we currently ship node modules as part of our SDK, we also need to ensure that we've npm installed dependencies for each module we ship. Since this process could end up pulling down platform specific dependencies, we need to do this for each platform we support.

Because we have a bunch of repositories, simply combining builds from each one could lead to problems, since we haven't validated the bits work well together. Because of this, we use `pulumi-cloud` to help us determine what components to use to produce an SDK.

We build the SDK by automatically queuing builds to the [pulumi/sdk](https://github.com/pulumi/sdk) repository after every green build job in pulumi-cloud (see [queue-sdk-build.sh](https://github.com/pulumi/pulumi-cloud/blob/master/scripts/queue-sdk-build.sh)), that build ends up calling [build-sdk.sh](https://github.com/pulumi/sdk/blob/master/scripts/build-sdk.sh) passing a commit hash for `pulumi-cloud` and the version tag we used for `pulumi-cloud`. These are passed to the `build-sdk` scripts.

The script clones `pulumi-cloud` at the specific commit, then use the `Gopkg.lock` file to determine the versions of all the dependencies to include in the SDK. Doing this, we are certain that our SDK contains the same bits we just validated in `pulumi-cloud`.  This SDK is published to our `rel.pulumi.com` S3 bucket in the production account.

## Building an SDK manually

You should never need to do this, since we always queue builds automatically for each pulumi-cloud build. If the SDK builds fail, you can just have Travis or AppVeyor rebuild. You could also run `./scripts/queue-sdk-build.sh` manually from `pulumi-cloud` (you'll need your Travis API Key (you can get this from the Travis UI or by running `travis token --pro` locally if you've installed the travis CLI) and [AppVeyor API Key](https://ci.appveyor.com/api-token) to do so).

If you find yourself having to produce SDKs manually, we should figure out why that is and add automation so you don't have to.

## Listing the release and updating documentation

In order for customers to get the new release, you need to update the [install page on docs.pulumi.com](https://docs.pulumi.com/install/). Note that the /releases/ endpoint on `docs.pulumi.com` will proxy any requests for non-prerelease published SDKs to the right S3 bucket.

1. In your `pulumi/docs` repository, update [`install/index.md`](https://github.com/pulumi/docs/blob/master/install/index.md). Update the variable `installer_version` in the YAML [front matter](https://jekyllrb.com/docs/frontmatter/). Verify that the links are correct when you generate the site locally.
2. Generate a new changelog, following the instructions in the docs repo readme [Generating a change log](https://github.com/pulumi/docs#generating-a-change-log).
3. Update the quickstart and reference content to integrate any breaking changes. Add or remove known issues from [known-issues.md](https://github.com/pulumi/docs/blob/master/reference/known-issues.md).
4. Update the READMEs and `package.json` in [Pulumi examples](https://github.com/pulumi/examples). 
5. Generate new API reference docs, following the instructions in https://github.com/pulumi/docs#development. 
6. Edit [dashboard.json](https://github.com/pulumi/docs/blob/master/dashboard.json) to add the new release in the Console dashboard "News" card.
7. Merge your changes into master. Then, create a new PR to merge into production.
8. To push the new content live, follow the instructions in [Updating Website Content](https://github.com/pulumi/home/wiki/Updating-the-Docs-Website#updating-website-content).