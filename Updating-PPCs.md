The first rule of updating PPCs is _don't_. If the update somehow wedges the Pulumi program, it's possible that a customer would be in a world of hurt. But if you know what you are doing, read on.

For Pulumi Service repositories, this is all found in `scripts/ops/update-service-ppcs.sh`.

Things to consider:

Which version of the PPC do you want to use? When updating the PPC we pick up whatever version is checked into the `Gopkg.lock` file in the `pulumi-service` repo. You might need to run `dep ensure --update github.com/pulumi/pulumi-ppc` first.

Do you want to edit the configuration of the PPC? If you need to change some configuration data, run the following. Be sure to add `--save` and `--stack`! Otherwise your config changes will only be stored locally, and the next person to update them won't have the changes. (A corollary is that you'll want to commit the updated `Pulumi.yaml` file too.)

## Standing up a new PPC

To stand up a new PPC, first you need a new AWS account to run it in. Second, you need to create a dedicated IAM role in that account which is _used to deploy the PPC_. (But will have nothing to do with the applications the PPC deploys.)

### Prerequisites

Creating a new AWS account and the IAM user role `pulumi-ppc` is mostly automated. See [Google Sheet](https://docs.google.com/spreadsheets/d/1ASpyMHUvC1rCN_6cRP6tq1D3378YzSC0PlHzvv_G42I/edit?ts=5a1c642f#gid=840536896) and [pulumi-service/scripts/opts/configure-ppc-account.sh](https://github.com/pulumi/pulumi-service/blob/master/scripts/ops/configure-ppc-account.sh).

### Configuration

The current way to configure a PPC (default regions, credentials, etc.) is TBD. But the min-bar of configuration is to know which AWS region the PPC will run in.

### Stack Creation

When you are ready to standup your new PPC, check out a new feature branch of `pulumi-service`. We'll need to commit our new PPC's config info in a `Pulumi.yaml`file in that repo.

```bash
cd-pulumi-service
git checkout -b standup-new-ppc

# Use the AWS credentials of the IAM `pulumi-ppc` user configured for the PPC's containing
# AWS account. (See above.)
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."

# Additional environment variables that are required to be set.
# Pass phrase used for secretes in Pulumi.yaml. See "Shared Passwords" doc on Google Drive.
export PULUMI_CONFIG_PASSPHRASE="..."
# Use shorter S3 bucket names.
export PULUMI_USE_POPS_S3_BUCKET="true"

# The PPC's stack name will be of the form "learningmachine-ppc-malta".
# The prefix comes from PULUMI_STACK_NAME_OVERRIDE. The suffix comes from
# a command-line parameter to create-ppc.sh.
#
# The following stands up "learningmachine-ppc-malta" in "eu-west-1".
export PULUMI_STACK_NAME_OVERRIDE="learningmachine"
./scripts/ppc/create-ppc.sh malta eu-west-1
```

Hopefully the PPC Pulumi program updated successfully. When finished, be sure to do three things:

1. Write down the PPC's API endpoint (something like https://xxxx.execute-api.eu-west-1.amazonaws.com/stage/) and access token (something like `AWUhfZ7tfdHFgZAAhmU9`). From the program's config and resource output. These are the values you need later.
2. Commit the updated `scripts/ppc/Pulumi.yaml` and send a PR so it gets checked in. Otherwise, nobody else can update your PPC.
3. Attach the PPC to the Pulumi Service. The final step is to attach the PPC to a Pulumi Organization on the web UI. To do so however, you must be an administrator of the organization.

## Updating

Updating the PPC requires some undocumented and poorly explained environment variables. We'll make these the default after the M9 bits shake out.

```bash
# Change config for stack "testing-moolumi-ppc-default"

# Step 1: Download the latest PPC package. (You need to do this before changing the AWS creds.)
./scripts/ppc/sync-ppc-package.sh

# Step 2: Set your current AWS keys to those from the account that owns the PPC.
export AWS_ACCESS_KEY_ID="AKIAJ2OH7AXVDPKFM4UA"
export AWS_SECRET_ACCESS_KEY="..."

# Step 3: Update PPC "testing-moolumi-ppc-default".
#         Notice that the PPC name has two parts "testing-moolumi" and PPC suffix "default":
export PULUMI_STACK_NAME_OVERRIDE="testing-moolumi"
export PULUMI_CONFIG_PASSPHRASE="<in shared passwords doc on Google Drive>"
export PULUMI_USE_POPS_S3_BUCKET="true"
./scripts/ppc/update-ppc.sh default
```

Note that there is a spreadsheet with PPC account info at https://docs.google.com/spreadsheets/d/1ASpyMHUvC1rCN_6cRP6tq1D3378YzSC0PlHzvv_G42I, and the shared passwords doc is at https://docs.google.com/document/d/1qetreL_sCvRVHAQildw-z3AkXFvg1AKowsxKm2G2h4M.