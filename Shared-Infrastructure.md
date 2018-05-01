We have a small number of pieces of infrastructure that are shared between accounts, and are not managed by any Pulumi program.

* The [`rel.pulumi.com`](https://s3.console.aws.amazon.com/s3/buckets/rel.pulumi.com/) bucket in the production account stores releases. Server Access Logging is enabled, and access logs are sent to the [`rel-access-logs`](https://s3.console.aws.amazon.com/s3/buckets/rel-access-logs/) bucket. 
* The [`eng.pulumi.com`](https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/) bucket in the dev account stores test run data.
* CloudTrail logging in each account - defined by https://github.com/pulumi/home/tree/master/infrastructure/cloudtrail-logging.