Though we update PPCs as part of our weekly rollout, the process is not *yet* as straightforward or reliable as we would like.

Please read carefully and file issues for any sharp corners.

---

## Updating Service PPCs

There are current (as of 1/18) two "types" of PPCs: those that are updated with the service instance, and those in a "customer-production" bucket.

### Service-dependent PPCs

Each service environment has a PPC we use for testing attached to the `Moolumi` organization. This is updated along with the Pulumi service for each environment. The production service environment also has a PPC attached to the `Pulumi` organization too, which is also updated along with the service.

These PPCs will automatically be updated whenever a commit is made to the `master`, `staging`, or `production` branches. Specifically, the operations are:

1. `make deploy`
1. `scripts/update-environment.sh`
1. `scripts/update-ppcs.sh`

### Customer-Production PPCs

PPCs that our customers use are _not_ (yet) updated automatically. Updates to the PPC used to cause downtime and require coordinating with customers. PPCs now remain available during updates, but the manual process remains as a historical artifact.

To update customer PPCs:

1. Get a Travis API token.
   - Log in to the [Travis API Explorer](https://developer.travis-ci.com/explore/#explorer) with your GitHub account and copy the token that appears (blacked-out) after `Authorization: token` in the Explorer view.
   - Or, if you have the [Travis CLI](https://github.com/travis-ci/travis.rb) installed, you can run `travis token --pro` to print your access token for `travis-ci.com`.

   Make sure not to confuse <tt>travis-ci<b>.org</b></tt> and <tt>travis-ci<b>.com</b></tt> ("Travis Pro"). They're two different services that require different API keys. Here, you should be using a token for Travis Pro.

   The link to the API explorer above goes to `travis-ci.com`. On the command line, use `--org` or `--pro` to choose the endpoint for one command, or `travis endpoint --set-default` with `--org` or `--pro` to set the default for later commands.
   
2. Run `scripts/ops/launch-update-customer-ppcs-job.sh $your_travis_token`. This launches a Travis job to perform the update. This runs `scripts/ops/update-ppcs.sh production_customer` as part of the `travis_api` make target.
3. Find the launched Travis job in the [list of jobs for `pulumi-service`](https://travis-ci.com/pulumi/pulumi-service). The job will be named `Update PPCs (production_customer)`. As for service releases, put a link to this Travis job in [`#releases`](https://pulumi.slack.com/messages/C79MDKGMV/) and follow [`#ops-notifications`](https://pulumi.slack.com/messages/C8FNQFZQQ/) and the Travis job for update progress.

## Standing up a new PPC

For Pulumi Service repositories, this is all found in `scripts/ops/update-service-ppcs.sh`.

Things to consider:

- Which version of the PPC do you want to use? When updating the PPC we pick up whatever version is checked into the `Gopkg.lock` file in the `pulumi-service` repo. You might need to run `dep ensure --update github.com/pulumi/pulumi-ppc` first.

- Do you want to edit the configuration of the PPC? If you need to change some configuration data, run the following. Be sure to add `--save` and `--stack`! Otherwise your config changes will only be stored locally, and the next person to update them won't have the changes. (A corollary is that you'll want to commit the updated `Pulumi.yaml` file too.)

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

The following details how to update a PPC by hand. This is what you'll need when maintaining a "dev stack" PPCs. For updating PPCs that are associated with an instance of the Pulumi Service (e.g. one we use for a production environment and/or customer PPC, see the "Updating Service PPCs" section below.)

First, be careful that you are using the correct version of the CLI.  If you aren't, you may end up compiling and deploying the program using an incorrect version of the CLI, which may cause unexpected failures.

Now that you've done that, here are the steps to run, in order:

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

## Maintenance Mode

PPCs can be put into "maintenance mode", which will return 503 "Service Unavailable" for any request to create stacks, update stacks, etc.

There is no way to do this through the UI. It must be done by directly editing the database. Each PPC (or row in the `Clouds` table) has a `flags` column that contains bit flags for various status.

The maintenance mode flag is the 0x1 bit.

### Database Access
First you need to open up the MySQL prompt for the given service database.

- Open up [this doc](https://docs.google.com/document/d/1p1jvOxbyiy_2OYdtWMjgY3W7vVZXx3Hq3O8Eq1Euea8/edit#) and set the `AWS_SECRET_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables for the environment you wish to update. (e.g. user `travis-cicd` in the production account.)
- Set `PULUMI_STACK_NAME_OVERRIDE` to "production", "staging", or "testing" depending on the service environment you want to update.
- Run `[pulumi-service] ./scripts/launch-mysql-prompt.sh`.

### Setting Maintenance Mode

```
# List the Clouds you are interested in updating. Note the current flag values.
# the FAKE-PPC-FOR-TESTING has 0x2 set (opt out of health checks).
SELECT name, id, org_id, flags FROM Clouds;
+----------------------+--------------------------------------+--------------------------------------+-------+
| name                 | id                                   | org_id                               | flags |
+----------------------+--------------------------------------+--------------------------------------+-------+
| FAKE-PPC-FOR-TESTING | 2632f283-c2f5-434f-a8ac-1e8bb54c8deb | feacc792-460f-4525-a091-e8de1f6ef34c |     2 |
...
```

```
# Set all PPCs associated with a given Organization (e.g. pulumi) to be in maintenance mode.
# Use the org_id value you saw above.
UPDATE Clouds SET flags = 1 WHERE org_id = "...";
```

### Clearing Maintenance Mode

To take a PPC out of maintenance mode, just remove the 0x1 bit from the `flags` column. The 99.9% case is just setting it to be 0. But if the PPC had some special situation (e.g. other `flags` set) be sure to restore them as needed.

```
# Clear the "maintenance mode" flag for PPCs.
UPDATE Clouds SET flags = 0 WHERE org_id = "...";
```