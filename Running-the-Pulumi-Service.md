# Running the Pulumi Service

This page covers the ins and outs of running the Pulumi Service, it's various production environments, and the generate care and feeding of the product.

The Pulumi Service runs in three different production environments: _testing_, _staging_, and _production_. We have a special GitHub organization just for testing the Pulumi Service, https://github.com/moolumi.

|   | Testing | Staging | Production |
|---|---|---|---|
| Used for...  | Sanity checking | Bleeding edge  | Keeping customers happy  |
| Located at...| https://beta-dot-testing.moolumi.io | https://beta.moolumi.io | https://beta.pulumi.com |
| Updated on commits to...  | `master`  | `staging`  | `production`  |
| PPCs for...  | Moolumi  | Moolumi, Pulumi  | Moolumi, Pulumi   |

You may optionally stand up your own _development_ instance of the Pulumi Service, e.g. https://beta-dot-chris.moolumi.io. Details on that later.

## AWS Account Information

The most important thing to keep (as it will be a source of bugs and confusion) is that each environment runs in a different AWS account. Summary:

|   | Acct ID | Account Email |
|---|---|---|
| **development** | 153052954103 | joe@pulumi.com |
| **testing** | 086028354146 | aws-contact+testing@pulumi.com |
| **staging** | 098437015098 | aws-contact+staging@pulumi.com |
| **production** | 058607598222 | aws-contact+production@pulumi.com |

For the rest of this guide it is assumed that you have configured your AWS console so that you can [switch between accounts](https://docs.google.com/document/d/1Do4YHOQSM6yxnXVef0dcsZ_8sqpOLm4w6Tri0KfzUFM/edit#bookmark=id.kx5t84v0ruxn), so you can view logs, etc.

You can find more information about our production accounts in:

- [AWS Root Accounts](https://docs.google.com/document/d/1Do4YHOQSM6yxnXVef0dcsZ_8sqpOLm4w6Tri0KfzUFM/edit)
- [AWS Role Account Passwords](https://docs.google.com/document/d/1p1jvOxbyiy_2OYdtWMjgY3W7vVZXx3Hq3O8Eq1Euea8/edit#heading=h.91hjjlhh9t63)

## Deployment

This section covers how the Pulumi Service is deployed. Updating a production environment should be handled by Travis as part of our CI/CD pipeline, however things can break down from time to time.

The Pulumi Service is deployed using the `pulumi` CLI, using the "local" mode of operation. (i.e. we don't require the Pulumi Service / PPC to deploy the Pulumi Service.) The scripts for managing a Pulumi Service stack is found in the [scripts](https://github.com/pulumi/pulumi-service/tree/master/scripts) folder.

### Travis Integration

When Travis (https://travis-ci.com) starts an automated build, it runs a Makefile target based on the type of job. e.g. `travis_push`. If you look at the Makefile, you'll notice that if we are executing a `travis_push` job and pushing into the master, staging, or production branch, then we run the `make deploy` target, which in-turn calls `scripts/update-environment.sh`. This is how the Pulumi Service environment gets updated.

`update-environment.sh` calls `pulumi update` as needed. The key thing to note however is _who_ the update is being ran as. Travis is configured to run as a role account within the "testing" AWS account, user `travis-cicd`. But in order to update the staging or production environments, the Travis script needs to switch AWS account.

This is done in the `scripts/set-travis-env.sh` script, which is called when Travis is fist starting. (See `before_install` in `.travis.yml`.) That script changes the values of the environment variables needed to update the Pulumi Stack, based on the branch being deployed into.

### Updating Staging and Production

We deploy to `staging` and `production` environments directly from the `staging` and `production` branches.

To update `staging`, ensure that `master` is green, then create a PR between `master` and `staging`.  Once that PR is green and approved, make sure that you merge with the "create a merge commit" option.

Similarly, to update `production`, ensure that `staging` is green, then create a PR between `staging` and `production`.  Once that PR is green and approved, make sure that you merge with the "create a merge commit" option.

### Pulumi CLI Integration

We don't use stack `pulumi update` for managing the Pulumi Service stacks, we go out of our way to ensure that two different people can update a production environment without any problems. So we do two things to ensure this:

1. Check all configuration into source code control. You can store Pulumi configuration within a `Pulumi.yaml` or keep it on your local machine in the `.pulumi` folder. All configuration _must_ be in `Pulumi.yaml`, otherwise the next update won't have the local configuration on another developer's machine.
1. We mirror local checkpoint files to S3. This way `pulumi` can compute the proper resource diff even if the previous deployment was ran on another machine.

## The Pulumi Service Database

All of the data for the Pulumi Service (e.g. user accounts, known GitHub organizations, configured PPCs and Stacks, etc.) are all housed in an [Amazon RDS](https://aws.amazon.com/rds/) instance. (See [infrastructure/database.ts](https://github.com/pulumi/pulumi-service/blob/master/infrastructure/database.ts).)

The mechanics for how the root password is set and migrations are ran can be found in [doc/db.md](https://github.com/pulumi/pulumi-service/blob/master/doc/db.md). For troubleshooting production-related issues, the script to be aware of is `scripts/launch-mysql-prompt.sh`.

Running `launch-mysql-prompt.sh` will start the MySQL console connected to your Pulumi Service database. The specific stack's database is determined by your current environment variables. (`PULUMI_STACK_NAME_OVERRIDE`, the current AWS account ID, etc.)

From there you can run various queries to inspect the state of data. **Be careful**. It is crazy-easy to break things by running an errant query. So unless you know what you are doing, avoid any stateful queries. And even then, you probably shouldn't.
