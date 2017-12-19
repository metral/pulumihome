We use Git to perform deployments and branches to represent our distinct running environments.

There are three important branches in the [`pulumi/pulumi-service`](https://github.com/pulumi/pulumi-service) repo, each corresponding to one of our environments:

* [`master`](https://github.com/pulumi/pulumi-service/tree/master): our test environment
* [`staging`](https://github.com/pulumi/pulumi-service/tree/staging): our staging environment (https://beta.moolumi.io)
* [`production`](https://github.com/pulumi/pulumi-service/tree/production): our production environment (https://beta.pulumi.com/)

# Deployments

To perform a deployment, create a PR on GitHub targeting the environment you wish to update.

All changes eventually get merged into `master`.  This triggers an update of the test environment.

**To promote a change to staging**, after it has been verified in the testing environment _(TODO: describe the practical meaning of "verified")_, create a PR on GitHub where the "base" branch is `staging` and the "compare" branch is `production`.  It should be auto-mergeable:

![image](https://user-images.githubusercontent.com/3953235/33845199-870fb526-de57-11e7-9fc1-25488088b45f.png)

**To promote a change to production**, after it has been verified in the staging environment _(TODO: describe the practical meaning of "verified")_, create a PR on GitHub where the "base" branch is `production` and the "compare" branch is `staging`.  It should be auto-mergeable:

![image](https://user-images.githubusercontent.com/3953235/33845119-3bd6ae5c-de57-11e7-8767-655672a544a8.png)

Note that if the target environment is already up to date, you will see the following message:

![image](https://user-images.githubusercontent.com/3953235/33845081-1d64c2b0-de57-11e7-860e-bd342984cb13.png)

In the event that a change isn't auto-mergeable, it means we haven't followed this procedure.  Please ask Chris or Joe how to proceed.

The only time we should deviate from this process is when deploying a hotfix (in which case, please see below).

# Deploying a Hotfix

We do not yet have a standard deployment process outside of the above.  We will write this section as we develop one.