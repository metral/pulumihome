# Weekly release process

How does our code get to customers, and what happens along the way?

## Overview

Code in the Pulumi service moves from `master` to `staging` to `production`.

Pulumi's release process follows one-week cycles. On Thursday we choose a commit to promote from `master` to `staging`. If all goes well there, that release is promoted to `production` on Tuesday. If everything *doesn't* go well, we decide whether to cherry-pick fixes or abandon the release.

## Roles

Releases to `production` are handled by primary oncall.

Releases to `staging` are handled by secondary oncall. Their first task as primary the next week will be to promote that release to `production`, so it's helpful for them to have context on the `staging` payload.

In each case, oncall has the final say about whether to continue or abandon a release.

## Background: What gets updated?

Updating the Pulumi Service actually refers to several components. "Changed" means what would cause a new version of the component to be updated as part of the weekly update process.

- The Pulumi Service (web frontend, API backend). This is changed with every new commit to the `pulumi/pulumi-service` repository.
- PPCs attached to the Pulumi Service. This is changed via a new version hash to `pulumi/pulumi-ppc` in `pulumi-service/Gopkg.lock`.
- The Pulumi SDK being used to run `pulumi update` in fire-and-forget mode, against the Pulumi Service and PPCs. This is updated by editing the `pulumi-service/.travis.yml` file.

Whenever we run the "Weekly Update Process", we will potentially be rolling out a new version of each of these components. All of which pose a great risk to our users without careful testing and qualification.

### Pulumi SDK releases

New SDK releases are built on-demand using the [process outlined here](https://github.com/pulumi/home/wiki/Producing-an-SDK).  New SDKs may be built multiple times per week, or just a couple times per sprint, depending on the frequency with which we want to get new features, fixes, and improvements out to customers.  Part of the SDK updating process is also updating the hashes in consuming service repos.  So, by the time we get to the weekly update process, the SDK being used will have been determined for us.

### Pulumi PPC releases

Updates to the software running on PPCs are handled similarly to the SDK. The referenced commit hash may be updated several times per sprint, or per week depending on the frequency with which we want to pick up new features, fixes, etc.

## Background: Service environments

There are three "environments" of the Pulumi Service: testing, staging, and production.

The exact mechanics of source integration and testing done are described below, but in short:

- _testing_ is updated with every commit to `pulumi-service` that gets merged into _master_.. The live site can be accessed at https://beta-dot-testing.moolumi.io.
- _staging_ is updated every morning, with the latest green build in _testing_. The live site can be accessed at https://beta.moolumi.io. (NOTE: This is a gosh darn lie. We haven't automated promoting testing to staging nightly yet. But we plan on doing that soon.)
- _production_ is updated once a week, with the latest green build from _staging_. The live site can be accessed at https://beta.pulumi.com.

## Code Promotion Process

The Pulumi Service updating process is (ab)uses Travis. Every update or test step is performed as part of a Travis job. Because of this, it is important to know when and how Travis jobs are triggered.

There are four times of Travis jobs: `push`, `pull_request`, `cron`, and `api`.

A `pull_request` job is created whenever a GitHub pull request is created or source branch is pushed to. We use `pull_request` jobs where the source branch is { _random feature branch_, `master`, or `staging` } to perform testing of changes before we merge them into { `master`, `staging`, or `production` }. Note that the service environment will _not_ be updated as part of this Travis job; we just test the changes.

A `push` job is created whenever a Git branch is updated. This is where we update the Pulumi Service environment to reflect what has been checked in. For example, whenever code gets merged into the `master` branch, the corresponding Travis `push` job runs `pulumi update` on the service in the testing environment and its corresponding PPC(s).

One important thing that falls out of this, is that in order to "roll back" a change you would need to trigger a new `push` job. (e.g. something like `git checkout HEAD^1 --hard && git push origin master --force`.) However, that might not work. If the Pulumi SDK used to deploy the service was updated as part of the commit, rolling back to an older Pulumi SDK might have consequences. (See [#538](https://github.com/pulumi/pulumi-service/issues/538))

An `api` job is one which is triggered using the Travis API or CLI. We use Travis `api` jobs to perform specialized operations or long-running tests. For example, to update customer PPCs we may trigger a new `api` job to run a different set of Makefile targets than typical `push` jobs run.

### Testing to Staging

Every night at 2AM, the service repo attempts to promote the `master` (_testing_ environment) branch to `staging` (_staging_ environment). This is done by creating a GitHub pull request, which is done automatically via tool. (Again, this is a darn line. But hopefully this will be the case soon. For now, you need to manually promote testing to staging.)

To do this manually, run the following command. It requires you have a GitHub access token stored in the  `GITHUB_ACCESS_TOKEN` environment variable.

```bash
# Promote the code from the master branch to staging.
cd ${GOPATH}/src/github.com/pulumi/home
./scripts/promote-release.sh master staging
```

The pull request triggers various Travis jobs, which if they all pass, confirm the bits are good and ready to be promoted. We run a few long-running integration tests as part of pull requests for promoting code between environments. So be sure to keep an eye on the Travis jobs in case they fail and/or need some attention.

Once the `pull_request` job has successfully completed, go ahead and merge the PR. (**Important** select the option to create a merge commit, do **not** rebase or squash the commits.)

Once the pull request is merged, a `push` job will be triggered which causes Travis to actually update the staging environment with the new changes. The steps performed are [documented here](https://github.com/pulumi/home/wiki/Updating-the-Service).

### Staging to Production

Every Tuesday at 2PM, the on-call primary selects the latest green candidate deployment from `staging`, and promotes it to the `production` branch (_production_ environment), using the steps used for promoting "Master to "Staging".

```bash
cd ${GOPATH}/src/github.com/pulumi/home
./scripts/promote-release.sh staging production
```

Like before, we run a lot of additional tests in the pull request job. So it will take a while to complete.

#### Customer PPCs

Note that updating customer PPCs is a separate activity. After the Pulumi Service has been updated, and rolled out to production. (i.e. the PR from `staging` to `production` has successfully been merged and tested.) We then able to update customer PPCs.

The process of updating customer PPCs [is documented here](https://github.com/pulumi/home/wiki/Updating-PPCs). (Once we've worked out all the kinks and have enough testing, we can add this as an automatic step.)

Once all Customer PPCs have been updated, the release is complete.