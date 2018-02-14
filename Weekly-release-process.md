How does our code get to customers, and what happens along the way?

## Overview

Code in the Pulumi service moves from `master` to `staging` to `production`.

Pulumi's release process follows one-week cycles. On Thursday we choose a commit to promote from `master` to `staging`. If all goes well there, that release is promoted to `production` the following Tuesday. If everything *doesn't* go well, we decide whether to cherry-pick fixes or abandon the release.

## Roles

Releases to `production` are handled by primary oncall.

Releases to `staging` are handled by secondary oncall. Their first task as primary the next week will be to promote that release to `production`, so it's helpful for them to have context on the `staging` payload.

In each case, oncall has the final say about whether to continue or abandon a release.

## Background: What gets updated?

Updating the Pulumi Service actually refers to several components. "Changed" means what would cause a new version of the component to be updated as part of the weekly update process.

- The Pulumi Service (web frontend, API backend). This is changed with every new commit to the `pulumi/pulumi-service` repository.
- PPCs attached to the Pulumi Service. This is changed via a new version hash to `pulumi/pulumi-ppc` in `pulumi-service/Gopkg.lock`.
- The Pulumi SDK being used to run `pulumi update` in fire-and-forget mode, against the Pulumi Service and PPCs. This is updated by editing the `pulumi-service/.travis.yml` file.

Whenever we run the "Weekly Update Process", we will potentially be rolling out a new version of each of these components.

### Pulumi SDK releases

New SDK releases are built on-demand using the [process outlined here](https://github.com/pulumi/home/wiki/Producing-an-SDK).  New SDKs may be built multiple times per week, or just a couple times per sprint, depending on the frequency with which we want to get new features, fixes, and improvements out to customers.  Part of the SDK updating process is also updating the hashes in consuming service repos.  So, by the time we get to the weekly update process, the SDK being used will have been determined for us.

### Pulumi PPC releases

Updates to the software running on PPCs are handled similarly to the SDK. The referenced commit hash may be updated several times per sprint, or per week depending on the frequency with which we want to pick up new features, fixes, etc.

## Service environments

The Pulumi service is deployed into three different environments:

- `testing` at https://beta-dot-testing.moolumi.io
- `staging` at https://beta.moolumi.io
- `production` at https://beta.pulumi.com

The `staging` and `production` environments track the corresponding branches in [pulumi/pulumi-service](https://github.com/pulumi/pulumi-service). The `testing` environment tracks `master`.

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

## Rolling back releases

If a critical bug is deployed to one of our environments, our current best response is to push *new* code to fix it -- which might mean fixing the bug, or might mean reverting the commit with the bug or the merge commit that brought the bug in.

We know we need a [much better story](https://github.com/pulumi/pulumi-service/issues/538) here.

## Appendix: The different kinds of Travis jobs

Since we use Travis for deployments and verification, it's helpful to know the different kinds of jobs that can run on Travis and how they're triggered:

- A `pull_request` job is created whenever a GitHub pull request is created or source branch is pushed to. We use `pull_request` jobs to check that the code builds and passes simple tests.

- A `push` job is created whenever a Git branch is updated. We use `push` jobs to deploy from `master`, `staging`, and `production` and to run longer tests after deployment.

- A `cron` job runs on a schedule, say `daily` or `weekly`.

- An `api` job is triggered using the Travis API or CLI. We use Travis `api` jobs to perform specialized operations or long-running tests. For example, to update customer PPCs we may trigger a new `api` job to run a different set of Makefile targets than typical `push` jobs run.