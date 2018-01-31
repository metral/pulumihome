The Pulumi Service's stacks -- such as `testing`, `staging`, and `production` -- are managed using the Pulumi CLI in fire-and-forget mode.  This is why we use bash scripts and S3 buckets to manage them, rather than the significantly easier workflow our customers get when they use the Pulumi SaaS product for managing their stacks.

Being fire-and-forget based, there are times when you need to interact with the stacks, for instance to set configuration variables or inspect the checkpoint state to find certain values (like load balancer URLs), and will need to synch state to your workspace in order to do so.

:warning: **Be very careful with the downloaded checkpoint file, as any modifications you make will not be reflected in the source of truth kept in S3.  This is generally only save to use for read-only operations.**

To synch a Pulumi Service stack's checkpoint to your local workspace, there is [a handy `scripts/describe-stack.sh` script](https://github.com/pulumi/pulumi-service/blob/master/scripts/describe-stack.sh) that will download the checkpoint file while also giving you information about the load balancer and API DNS addresses.

To run this script, you must override the `AWS_DEFAULT_PROFILE` and `PULUMI_STACK_NAME_OVERRIDE` variables.  First, make sure you have configured AWS profiles for [our distinct accounts per environment](https://docs.google.com/document/d/1Do4YHOQSM6yxnXVef0dcsZ_8sqpOLm4w6Tri0KfzUFM/edit).  Let's assume we are interacting with the `testing` stack and that you have saved the associated AWS profile also as `testing`.  Then, this command will do the trick:

```
$ AWS_DEFAULT_PROFILE=testing PULUMI_STACK_NAME_OVERRIDE=testing ./scripts/describe-stack.sh
```

The output will be something like

```
# AWS_REGION: us-west-2
# ALB DNS   : albd38f9b81-912138058.us-west-2.elb.amazonaws.com
export PULUMI_API="https://api-dot-testing.moolumi.io"
```

and you will then be able to find the resulting `.pulumi/stacks/pulumi-service/testing.json` checkpoint file.