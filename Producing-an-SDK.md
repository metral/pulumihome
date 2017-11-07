# Producing an SDK

## Overview

When Travis or AppVeyor build any branch (or tag) successfully, they run our publishing scripts to upload binaries to [s3://eng.pulumi.com/releases](https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/releases/?region=us-east-1#).

We publish each build under three different names:
1. The full SHA of the commit we are building
2. The name of branch we are building (e.g. master)
3. The value returned by `git describe --tags` which is a name based on the newest tag reachable from the commit we are building.

Each repository produces an archive that is designed to be extracted into a root folder (`PULUMI_DIR`) which is the install location of Pulumi on a end user machine.

To create an SDK, we need to combine builds from all the branches we care about and then archive the combined build. Because we currently ship node modules as part of our SDK, we also need to ensure that we've npm installed dependencies for each module we ship.

We have scripts named `build-sdk.sh` and `build-sdk.cmd` in [scripts/](https://github.com/pulumi/home/tree/master/scripts) that automate this process.  Because the scripts run npm install, we need to run them on each OS we build for.

## Producing a new release
1. Tag commits for each repository with a version number (`git tag v0.8`) and push the tags to GitHub `git push origin --tags`). The scripts expect that every repository have the same tag, so if you are producing a hotfix, you'll still need to tag all the unchanged repositories with the new version.
2. CI will queue builds based on these tags and publish artifacts. If you are tagging existing commits that have been built before, you can continue, otherwise wait for these jobs to complete. Note that the version information we include in `pulumi` itself is based on the tag, so you should wait for `pulumi/pulumi` to build the tag so the resulting binary has the correct version.
3. Build the SDKs. The `build-sdk` scripts take two arguments: A version string to include in the produced archive's filename and a git revision to build. For the second argument, you can pass a branch name or a tag name, but that name needs to be the same across `pulumi/pulumi`, `pulumi/pulumi-aws` and `pulumi/pulumi-cloud`. The first argument defaults to the current date and time and the second argument defaults to master. When releasing milestone bits, you'll use something like `v0.8` for both (e.g. `./build-sdk.sh v0.8 v0.8`)
4. After you've run the script on macOS, Linux and Windows, the builds should be present in [s3://eng.pulumi.com/releases/sdk/](https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/releases/sdk/?region=us-east-1&tab=overview) folder on S3.

## Updating the docs website
Once you've made a new release, you'll probably want to update [docs.pulumi.com](https://docs.pulumi.com/) to have the new build. That's easy as well:

1. In your `pulumi/docs` repository, navigate to the `releases` folder. Make sure you have [git lfs](https://git-lfs.github.com/) installed, since we managed the binaries with git lfs.
2. Download the 3 SDK's you published to S3 (`scripts/download-sdks.sh` automates this using `aws s3 cp`).
3. Update `install/index.md` to have the new version number and links
4. Commit (when you commit git lfs will upload the binaries)
5. Push, then follow the directions in https://github.com/pulumi/pulumi-service/blob/master/cmd/docs/DEVOPS.md for information on how to regenerate the docs website.