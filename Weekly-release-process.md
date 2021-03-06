How does our code get to customers, and what happens along the way?

## Overview

Code in the Pulumi service starts in `master`, then moves to `staging` and `production`. We manage this flow by creating **releases** from `master` and **promoting** releases to `staging` and then `production`.

## Schedule

We aim to promote code to each environment twice a week. Each Tuesday and Thursday, the release in `staging` is promoted to `production` and a new `staging` release is cut from `master`.

## Roles

Releases to `staging` and `production` are handled by primary oncall. If something goes wrong, they're in the best position to respond and they'll have context on what's changed.

Oncall has the final say about whether to continue or abandon a release.

## Background: What gets updated?

Updating the Pulumi Service actually refers to several components. "Changed" means what would cause a new version of the component to be updated as part of the weekly update process.

- The Pulumi Service (web frontend, API backend). This is changed with every new commit to the `pulumi/pulumi-service` repository. The mechanics of the service deployment are described [here](https://github.com/pulumi/home/wiki/Updating-the-Service).
- The Pulumi SDK being used to run `pulumi update` in fire-and-forget mode, against the Pulumi Service. This is updated by editing the `pulumi-service/.travis.yml` file.

Whenever we run the "Weekly Update Process", we will potentially be rolling out a new version of each of these components.

### Pulumi SDK releases

New SDK releases are built on-demand using the [process outlined here](https://github.com/pulumi/home/wiki/Producing-an-SDK).  New SDKs may be built multiple times per week, or just a couple times per sprint, depending on the frequency with which we want to get new features, fixes, and improvements out to customers.  Part of the SDK updating process is also updating the hashes in consuming service repos.  So, by the time we get to the weekly update process, the SDK being used will have been determined for us.

## Service environments

The Pulumi service is deployed into three different environments:

- `testing` at https://www.pulumi-test.io
- `staging` at https://www.pulumi-staging.io
- `production` at https://www.pulumi.com

The `staging` and `production` environments track the corresponding branches in [pulumi/pulumi-service](https://github.com/pulumi/pulumi-service). The `testing` environment tracks `master`.

## Promoting code

### To `staging`

Each Tuesday and Thursday, primary oncall creates a release branch (e.g. `release/2018-01-01`) from the most recent green build in `testing`, then creates a pull request to merge that branch into `staging`.

We use a Slack [bot](https://github.com/pulumi/home/tree/master/infrastructure/platypull) for this. Every Tuesday and Thursday at 10AM PST (and 9AM during Daylight Savings Time, i.e., PDT.). The bot will post a message in the `releases` Slack channel with three possible actions. So you don't have to do anything other than respond to the message posted by the bot. If you would like it to create a release PR, then you simply press "Yes" and the bot will take it from there.

Here's a preview of one such interaction with the bot:

![screen shot 2019-01-08 at 13 13 40](https://user-images.githubusercontent.com/1466314/50859224-a0081200-1347-11e9-9f89-b3e1fa55d493.png)

![screen shot 2019-01-08 at 13 13 48](https://user-images.githubusercontent.com/1466314/50859227-a1393f00-1347-11e9-8726-fa345afe51a6.png)

When you don't want to create a release PR, simply respond by clicking the "No" button. This will result in a response from the bot confirming your action not to create a PR, which will look something like this:

![screen shot 2019-01-08 at 13 21 48](https://user-images.githubusercontent.com/1466314/50859505-65eb4000-1348-11e9-811b-316efb30ef14.png)

#### Problems with the Slack bot? You can still use the manual way to create a release PR for `staging`.

The source for the tool is located [here](https://github.com/pulumi/home/tree/master/cmd/newrelease).

1. [Create a GitHub personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/) with the **`read:org`**, **`repo`**, and **`user:email`** scopes. The tool itself needs the `repo` scope; the other scopes are [for Travis](https://docs.travis-ci.com/user/github-oauth-scopes/).
2. Put the token in an environment variable named `GITHUB_TOKEN`.
3. In a checkout of the `pulumi/home` repo:

    ```bash
    cd "$(go env GOPATH)/src/github.com/pulumi/home"
    dep ensure
    go install ./cmd/newrelease

    # Dry run. (Optional.)
    # Test API access to GitHub and Travis and show what commit would be used.
    newrelease  # assumes ~/go/bin is on the $PATH

    # Choose a commit, and create a release branch and PR.
    newrelease -createbranch=true
    ```

#### Submitting

Get the PR reviewed by your oncall partner. Triage and address any CI failures by following up with code authors and perhaps taking cherry-picks to the release branch.

Once the PR is ready -- where "ready" is oncall's judgment but at least means all CI failures are understood --
 submit using the GitHub interface. You should **ALWAYS** click "Create Merge Commit". The reason for that is to allow for cherry picks without merge conflicts if needed.

![image](https://user-images.githubusercontent.com/4029847/44553758-68bffe00-a6e3-11e8-8f71-4e80df4b6ce7.png)

**Do not delete the `release/*` branch.**

As a courtesy, give a heads-up in [`#releases`](https://pulumi.slack.com/messages/C79MDKGMV/) with a link to the Travis job running the deployment. You should also take a quick look at the "badges" on the [dashboard for](https://github.com/pulumi/home/blob/master/infrastructure/stress-tester/README.md) the Stress Tester app, to ensure that the current pass rate is 100% against the testing environment. (i.e. there aren't any known problems.)

Once the PR is merged and the Travis job makes it to the head of the queue, the build+deploy takes about 30 minutes. Watch [`#ops-notifications`](https://pulumi.slack.com/messages/C8FNQFZQQ/), [`#builds`](https://pulumi.slack.com/messages/C5J0XFWRJ/), and [Travis](https://travis-ci.com/pulumi/pulumi-service) for updates on the build and deployment.


#### Out-of-date branches

Our release process creates merge commits in `staging` and `production` that will never themselves be merged back into `master`. As a result, GitHub will warn that release branches are out-of-date with the base branches.

To verify these warnings are benign, you can use GitHub's comparison view to check that `staging` and `production` only differ from `master` in merge commits:
- [Changes in `staging` that aren't in `master`](https://github.com/pulumi/pulumi-service/compare/master...staging#files_bucket)
- [Changes in `production` that aren't in `master`](https://github.com/pulumi/pulumi-service/compare/master...production#files_bucket)

#### Dealing with merge conflicts

The PR may warn that the branch can't merge cleanly. This can happen if we've taken cherry picks since the last release.

To make the PR mergeable, if it isn't already, first identify the *previous* release branch by looking at the [most recent merge to `staging`](https://github.com/pulumi/pulumi-service/commits/staging). Then check out the new release branch locally and merge in the old one:

```
git fetch  # make sure we have an up-to-date ref to the previous release branch
git checkout release/2018-02-15  # new release branch
git merge origin/release/2018-02-08 --strategy-option ours  # mark merged, prefer our (newer) code
git push
```

The resulting merge commit should have an **empty diff**.

### To `production`

#### Merge the changes
On Tuesday and Thursday, primary oncall merges the release branch that currently matches `staging` (e.g. `release/2018-02-12`) into `production`. We merge from the release branch, instead of from `staging`, so we can manage deployments (and cherry-picks) to each environment independently.

We don't have a tool for this yet, but you can create the PR from the GitHub UI or start with a URL like https://github.com/pulumi/pulumi-service/compare/production...release/2018-02-12.

Have the PR reviewed by your oncall partner and submit the same way you would the `staging` release.

Before merging however, take a quick look at the "badges" on the [dashboard for](https://github.com/pulumi/home/blob/master/infrastructure/stress-tester/README.md) the Stress Tester app, to ensure that the current pass rate is 100% against the staging environment. (i.e. there aren't any known problems.)

## Cherry-picks

To bring small fixes into the `staging` or `production` environment, cherry-pick commits **into the release branch** and then create another PR to merge the release branch back into the target environment(s). The PR to `staging` or `production` should be reviewed by primary or secondary oncall. The cherry-pick itself need not be.

```
git fetch
git checkout release/2018-03-01
# -m1 is necessary below if cherry-picking a merge commit
git cherry-pick -x abcdabcd -m1
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

- An `api` job is triggered using the Travis API or CLI. We use Travis `api` jobs to perform specialized operations or long-running tests.