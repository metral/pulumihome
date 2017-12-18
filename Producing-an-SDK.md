# Producing an SDK

## Overview

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

## Updating the docs website
Once you've made a new release and we've validate it, you'll probably want to update the [install page on docs.pulumi.com](https://docs.pulumi.com/install/). The /releases/ endpoint on `docs.pulumi.com` will proxy any requests for published SDKs without a prerelease tag to the S3, so you do not need to worry about uploading binaries anywhere.  However, you'll want to update the "latest" version.

1. In your `pulumi/docs` repository, update `install/index.md` to have the new version number. Set the `currentVersion` in the JavaScript section, and the links will be updated automatically.
2. Commit and then Push. (If you create a GitHub PR, be sure to merge that into `master` before proceeding to the next step.)
3. Follow the directions in https://github.com/pulumi/pulumi-service/blob/master/cmd/docs/DEVOPS.md for information on how to regenerate the docs website.