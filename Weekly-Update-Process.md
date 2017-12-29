We use a weekly update process to get bits out into production.  Aside from critical hotfixes (requiring manager approval), all deployments should follow this usual process.

The on-call primary is responsible for quarterbacking all weekly updates.  The on-call secondary shadows and assists as appropriate.

## Background: What gets updated?

Updating the Pulumi Service actually refers to several components:

- The Pulumi Service (web frontend, API backend). This is changed with every new commit to the `pulumi/pulumi-service` repository.
- PPCs attached to the Pulumi Service. This is changed via a new version hash to `pulumi/pulumi-ppc` in `pulumi-service/Gopkg.lock`.
- The Pulumi SDK being used to run `pulumi update` in fire-and-forget mode, against the Pulumi Service and PPCs. This is updated by editing the `pulumi-service/.travis.yml` file.

Whenever we run the "Weekly Update Process", we will potentially be rolling out a new version of each of these components. All of which pose a great risk to our users without careful testing and qualification.

### Pulumi SDK releases

New SDK releases are built on-demand using the [process outlined here](https://github.com/pulumi/home/wiki/Producing-an-SDK).  New SDKs may be built multiple times per week, or just a couple times per sprint, depending on the frequency with which we want to get new features, fixes, and improvements out to customers.  Part of the SDK updating process is also updating the hashes in consuming service repos.  So, by the time we get to the weekly update process, the SDK being used will have been determined for us.

Integration Test: "Test for upgrading the version of Pulumi SDK used"
https://github.com/pulumi/pulumi-service/issues/497

### Pulumi PPC releases

Updates to the software running on PPCs are handled similarly to the SDK. The referenced commit hash may be updated several times per sprint, or per week depending on the frequency with which we want to pick up new features, fixes, etc.

Integration Test: "Test for upgrading the version of `pulumi-ppc` used"
https://github.com/pulumi/pulumi-service/issues/496

## Background: Service environments

There are three "environments" of the Pulumi Service: testing, staging, and production.

The exact mechanics of source integration and testing done are described below, but in short:

- _testing_ is updated with every commit to `pulumi-service`. The live site can be accessed at https://beta-dot-testing.moolumi.io.
- _staging_ is updated every morning, with the latest green build in _testing_. The live site can be accessed at https://beta.moolumi.io.
- _production_ is updated once a week, with the latest green build from _staging_. The live site can be accessed at https://beta.pulumi.com.

## Code Promotion Process

### Testing to Staging

Every night at 2AM, the service repo attempts to promote the `master` (_testing_ environment) branch to `staging` (_staging_ environment). This is done by creating a GitHub pull request, which is done automatically via tool.

The pull request triggers various Travis jobs, which if they all pass, confirm the bigs are good and ready to be promoted. The process is [documented here](https://github.com/pulumi/home/wiki/Updating-the-Service)

Once the pull request is merged, a `push` job will be triggered which causes Travis to actually update the staging environment with the new changes.

### Staging to Production

Every Tuesday at 2PM, the on-call primary selects the latest green candidate deployment from `staging`, and promotes it to the `production` branch (_production_ environment), using the [same process as before](https://github.com/pulumi/home/wiki/Updating-the-Service).

#### Customer PPCs

Note that updating customer PPCs is a separate activity. After the Pulumi Service has been updated, and rolled out to production. (i.e. the PR from `staging` to `production` has successfully been merged and tested.) We then able to update customer PPCs.

The process of updating customer PPCs [is documented here](https://github.com/pulumi/home/wiki/Updating-PPCs).

Once all Customer PPCs have been updated, the release is complete.