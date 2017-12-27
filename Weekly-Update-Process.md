We use a weekly update process to get bits out into production.  Aside from critical hotfixes (requiring manager approval), all deployments should follow this usual process.

The on-call primary is responsible for quarterbacking all weekly updates.  The on-call secondary shadows and assists as appropriate.

New SDK releases are built on-demand using the [process outlined here](https://github.com/pulumi/home/wiki/Producing-an-SDK).  New SDKs may be built multiple times per week, or just a couple times per sprint, depending on the frequency with which we want to get new features, fixes, and improvements out to customers.  Part of the SDK updating process is also updating the hashes in consuming service repos.  So, by the time we get to the weekly update process, the SDK being used will have been determined for us.

Every night at 2AM, the service repo attempts to promote the `master` (Testing) branch to `stage` (Staging).  Travis will then update Staging and run smoke tests against it.  Any green build is called a "candidate" deployment.

Every Tuesday at 2PM, the on-call primary selects the latest green candidate deployment from Staging, and promotes it to `prod` (Production), using the [process outlined here](https://github.com/pulumi/home/wiki/Updating-the-Service).

Note that updating customer PPCs is a separate activity.  After performing the above, Travis will smoke test production.  After this is complete, the process of [updating customer PPCs may start](https://github.com/pulumi/home/wiki/Updating-PPCs).