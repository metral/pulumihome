The first rule of updating PPCs is _don't_. If the update somehow wedges the Pulumi program, it's possible that a customer would be in a world of hurt. But if you know what you are doing, read on.

For Pulumi Service repositories, this is all found in `scripts/ops/update-service-ppcs.sh`.

Things to consider:

Which version of the PPC do you want to use? When updating the PPC we pick up whatever version is checked into the `Gopkg.lock` file in the `pulumi-service` repo. You might need to run `dep ensure --update github.com/pulumi/pulumi-ppc` first.

Do you want to edit the configuration of the PPC? If you need to change some configuration data, run the following. Be sure to add `--save` and `--stack`! Otherwise your config changes will only be stored locally, and the next person to update them won't have the changes. (A corollary is that you'll want to commit the updated `Pulumi.yaml` file too.)

```
# Change config for stack "testing-moolumi-ppc-default"
# BUG? Note that the stack might not exist locally. You might need to run `update-ppc.sh` to download the
# checkpoint file so your local Pulumi doesn't get confused.
cd scripts-ppc
pulumi config set --save --stack=testing-moolumi-ppc-default key value
```

Updating the PPC requires some undocumented and poorly explained environment variables. We'll make these the default after the M9 bits shake out.

```
# Download the latest PPC package. (You need to do this before changing the AWS creds.)
./scripts/ppc/sync-ppc-package.sh

# Set your current AWS keys to be one from the testing account.
export AWS_ACCESS_KEY_ID="AKIAJ2OH7AXVDPKFM4UA"
export AWS_SECRET_ACCESS_KEY="..."

# Update PPC  "testing-moolumi-ppc-default"
# PPC name has two parts "testing-moolumi" and PPC suffix "default"
export PULUMI_STACK_NAME_OVERRIDE="testing-moolumi"
export PULUMI_USE_POPS_S3_BUCKET="true"
./scripts/ppc/update-ppc.sh default
```