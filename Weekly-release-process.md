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

## Promoting code

### To `staging`

On Thursday, secondary oncall creates a release branch (e.g. `release/2018-01-01`) from the most recent green build in `testing`, then creates a pull request to merge that branch into `staging`.

We use a [tool](https://github.com/pulumi/home/tree/master/cmd/newrelease) for this.

1. [Create a GitHub personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) with the **`read:org`**, **`repo`**, and **`user:email`** scopes. The tool itself needs the `repo` scope; the other scopes are [for Travis](https://docs.travis-ci.com/user/github-oauth-scopes/).
2. Put the token in an environment variable named `GITHUB_TOKEN`.
3. Run `newrelease`:

    ```bash
    go install github.com/pulumi/home/cmd/newrelease

    # Dry run. (Optional.)
    # Test API access to GitHub and Travis and show what commit would be used.
    newrelease -createbranch=false

    # Choose a commit, and create a release branch and PR.
    newrelease
    ```

#### Merging

The PR may complain that the branch can't merge cleanly. This can happen if we've taken cherry picks since the last release.

To make the PR mergeable, if it isn't already, first identify the *previous* release branch by looking at the [most recent merge to `staging`](https://github.com/pulumi/pulumi-service/commits/staging). Then check out the new release branch locally and merge in the old one:

```
git fetch
git checkout release/2018-02-15  # new release branch
git merge release/2018-02-08 --strategy-option ours  # mark merged, prefer our (newer) code
git push
```

The resulting merge commit should have an **empty diff**.

Get the PR reviewed by your oncall partner, triage and address any CI failure, then submit using the GitHub interface. Watch [`#ops-notifications`](https://pulumi.slack.com/messages/C8FNQFZQQ/), [`#builds`](https://pulumi.slack.com/messages/C5J0XFWRJ/), and [Travis](https://travis-ci.com/pulumi/pulumi-service) for updates on the build and deployment.

Once the CI jobs have successfully completed, go ahead and merge the PR. (**Important** select the option to create a merge commit, do **not** rebase or squash the commits.)

The mechanics of the service deployment are described [here](https://github.com/pulumi/home/wiki/Updating-the-Service).

### To `production`

On Tuesday, primary oncall merges the previous week's release branch into `production`.

We don't have a tool for this yet, but you can create the PR from the GitHub UI or start with a URL like https://github.com/pulumi/pulumi-service/compare/production...release/2018-02-12.

Send the PR to your oncall partner for review.

#### Customer PPCs

Note that updating customer PPCs is a separate activity. After the Pulumi Service has been updated, and rolled out to production. (i.e. the PR from `staging` to `production` has successfully been merged and tested.) We then able to update customer PPCs.

The process of updating customer PPCs [is documented here](https://github.com/pulumi/home/wiki/Updating-PPCs). (Once we've worked out all the kinks and have enough testing, we can add this as an automatic step.)

Once all Customer PPCs have been updated, the release is complete.

## Cherry-picks

To bring small fixes into the `staging` or `production` environment, cherry-pick commits **into the release branch** and then create another PR to merge the release branch back into the target environment(s). The PR to `staging` or `production` should be reviewed; the actually cherry-pick need not be.

```
git fetch
git checkout release/2018-03-01
git cherry-pick -x abcdabcd
git push
```

## Rolling back releases

If a critical bug is deployed to one of our environments, our current best response is to push *new* code to fix it -- which might mean fixing the bug, or might mean reverting the commit with the bug or the merge commit that brought the bug in.

We know we need a [much better story](https://github.com/pulumi/pulumi-service/issues/538) here.

## Appendix: The different kinds of Travis jobs

Since we use Travis for deployments and verification, it's helpful to know the different kinds of jobs that can run on Travis and how they're triggered:

- A `pull_request` job is created whenever a GitHub pull request is created or source branch is pushed to. We use `pull_request` jobs to check that the code builds and passes simple tests.

- A `push` job is created whenever a Git branch is updated. We use `push` jobs to deploy from `master`, `staging`, and `production` and to run longer tests after deployment.

- A `cron` job runs on a schedule, say `daily` or `weekly`.

- An `api` job is triggered using the Travis API or CLI. We use Travis `api` jobs to perform specialized operations or long-running tests. For example, to update customer PPCs we may trigger a new `api` job to run a different set of Makefile targets than typical `push` jobs run.