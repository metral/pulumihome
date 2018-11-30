# Producing an SDK

## Overview

The Pulumi SDK is made up of the CLI and language specific packages (e.g `@pulumi/pulumi`, `@pulumi/aws`, and `@pulumi/cloud`).  The CLI and packages ship independently from one another.  Because of this, we are able to release our individual packages independently.

## Releasing a new version of a package.

1. Ensure that the `CHANGELOG.md` in the root is up to date. You'll want to edit it to add a date to the version we are about to release and to add a new version above it that lets us track all the unreleased changes we'll do next.  If you are just adding the date, feel free to merge directly into master without review.

2. Tag the commit we you want to release with a tag'd version number. For example, if we are releasing v0.16.5 of a package, do the following (after you've merged in the `CHANGELOG.md` changes):

```sh
$ git checkout master
$ git tag v0.16.5
$ git push origin v0.16.5
```

If you'd like to release a build as a release candidate instead, append a `-rc.X` suffix to the TAG (where X is the RC number):

```sh
$ git tag v0.16.5-rc.1
```

When releasing an RC, you need not update the `CHANGELOG.md` file (since we have, of course, not released yet!)

### Releasing a hotfix for an existing package

In certain cases, we may not want to release `HEAD` of `master`, opting instead to cherry-pick a small number of fixes into a new release. Ideally, we'll always have `HEAD` of `master` in a good state, and following these directions will not be needed. The process for releasing a hotfix is a little more complex, but still managable.

1. Create a new branch, and rest it's `HEAD` to the most recently release version. For example, if we need to produce a hotfix for 0.16.4, we'd do the following:

```sh
git checkout -b ellismg/hot-fix-for-0.16.4
git reset --hard v0.16.4
```

2. Cherry-pick existing fixes from `master` into the new branch, or make new commits as needed.  When cherry-picking, include the `-x` flag, so the commit hash that was cherry-picked is appended to the commit message.

3. Update the `CHANGELOG.md` file to include information about the hotfixes and the release date. This file will now be diverged from `master`'s version, but we'll fix that up later.

4. Once you are ready to release, tag HEAD of your topic branch with the new version tag:

```sh
$ git tag v0.16.5 
```

Push your topic branch and tag to GitHub. Travis will kick off a build for the tag and publish the release once all tests pass. You can verify that the build completes in the Travis UI.

```sh
$ git push origin ellismg/hot-fix-for-0.16.4 v0.16.5
```

5. Open a PR to merge the topic back into master. This may have merge conflicts (that you need to resolve by first merging `master` into the topic branch. Note that you should do this merge **AFTER** you have tagged the commit you want to release. It is very likely you will have to do some manual edits of CHANGELOG.md after merging to ensure the file looks correct.

6. When merging the PR into master, ensure that you use the "CREATE MERGE COMMIT" option. Otherwise, the commit tag will be pointing at an orphaned commit. Also, the builds out of master will continued to be published as dev builds of the just released version (e.g. `v0.16.5-dev` instead of the new correct value, `v0.16.6-dev`)

## Listing the release and updating documentation

In order for customers to get the new release, you need to update the [install page on docs.pulumi.com](https://docs.pulumi.com/install/). Note that the /releases/ endpoint on `docs.pulumi.com` will proxy any requests for non-prerelease published SDKs to the right S3 bucket.

1. In your `pulumi/docs` repository, update [`install/index.md`](https://github.com/pulumi/docs/blob/master/install/index.md). Update the variable `installer_version` in the YAML [front matter](https://jekyllrb.com/docs/frontmatter/). Verify that the links are correct when you generate the site locally.
1. Generate new API reference docs, following the instructions in https://github.com/pulumi/docs#development. 
1. Edit [latest-version](https://github.com/pulumi/docs/blob/master/latest-version) to set the latest CLI version that the CLI will use for version checks. (The CLI will not request this directly, but through an API endpoint on the service that proxies requests to this file on the docs site.)
1. Install the latest `pulumi` CLI. Then, generate new CLI documentation by running `pulumi gen-markdown reference/cli` in the docs repo. 
1. Merge your changes into master. Then, create a new PR to merge into production.
1. To push the new content live, follow the instructions in [Updating Website Content](https://github.com/pulumi/home/wiki/Updating-the-Docs-Website#updating-website-content).

## Release Validation

The integration tests in the [home](https://github.com/pulumi/pulumi) and [examples](https://github.com/pulumi/examples) always test against the latest released packages. So you don't need to worry about updating either of these repositories. The nightly cron jobs also run the examples on the "dev" versions of packages (i.e. what is at HEAD of master of each repository).