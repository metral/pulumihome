This page lists many pointers and activities that are useful when doing live site investigations.  This includes correctness investigations in addition to performance.

# AWS

All of our services run in AWS.  You will need access.  Please see https://docs.google.com/document/d/1Do4YHOQSM6yxnXVef0dcsZ_8sqpOLm4w6Tri0KfzUFM for the various accounts and roles we use to manage the site.

# Environments

TODO: we need to document all of the environments, regions, accounts, etc.

# Investigating Failures

Our service uses CloudWatch for logging from all aspects of the service.  First login, then go to the region under investigation.  Search for the logs of interest.  TODO: known issue, the logs are not prefixed in an easy-to-search way, see https://github.com/pulumi/pulumi-service/issues/361.