The AWS SDKs and CLI let you switch between different named bundles of configuration called _profiles_. Profiles make it easier to use _assumed-role_ credentials.

You can read more about using profiles in the [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html).

## Setting up and using profiles

Let's say your `~/.aws/credentials` file has your user credentials:

```ini
[default]
aws_access_key = ABC
aws_secret_access_key = 123
```

Then you can add an entry to `~/.aws/config` to start with those credentials and assume another role -- one with more permissions or in a different account:

```ini
[profile pulumi-testing]
role_arn = arn:aws:iam::086028354146:role/OrganizationAccountAccessRole
source_profile = default

# Each profile has to specify a region. One might expect `region` or
# other settings to be inherited, but `source_profile` is only for
# specifying credentials to assume the IAM role.
region = us-west-2
```

You can find an example `config` file [below](#reference-awsconfig) with roles for many Pulumi accounts.

Now you can use that profile with the AWS CLI:

```
$ aws sts get-caller-identity
{
    "Account": "153052954103", 
    "UserId": "AIDAIKI6YAHYZ5XHFBF76", 
    "Arn": "arn:aws:iam::153052954103:user/mattdr@pulumi.com"
}

$ aws ecs list-clusters
{
    "clusterArns": [
        "arn:aws:ecs:us-west-2:153052954103:cluster/chris--1158885"
    ]
}


$ aws sts get-caller-identity --profile pulumi-testing
# or equivalently
# AWS_PROFILE=pulumi-testing aws sts get-caller-identity
{
    "Account": "086028354146", 
    "UserId": "AROAJWOEUDFPDZA6MI674:AWS-CLI-session-1523566057", 
    "Arn": "arn:aws:sts::086028354146:assumed-role/OrganizationAccountAccessRole/AWS-CLI-session-1523566057"
}

$ aws ecs list-clusters --profile pulumi-testing
{
    "clusterArns": [
        "arn:aws:ecs:us-west-2:086028354146:cluster/pulumi-testing-ppc-moolumi2-cluster-dc61184d18876689", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/pulumi-tst-up-172-global-067a5f9", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/pulumi-tst-up-214-global-15b321d", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/tst-up-27861--ba9cb76", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/pulumi-tst-up-244-global-2e2fe74", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/testing09861791", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/tst-up-24466--08d203a", 
        "arn:aws:ecs:us-west-2:086028354146:cluster/testing-7f3b7437"
    ]
}
```

### Using AWS profiles with `pulumi` or `terraform`

The AWS SDK for Go -- used by the Terraform and Pulumi AWS providers -- doesn't read `~/.aws/config` by default. Just setting `AWS_PROFILE` may lead to an error like:

```
error: failed to load resource plugin aws: failed to configure pkg 'aws' resource provider: No valid credential sources found for AWS Provider.
	Please see https://terraform.io/docs/providers/aws/index.html for more information on
	providing credentials for the AWS Provider
```

To make `AWS_PROFILE` work, you must also set `AWS_SDK_LOAD_CONFIG=1`. See "Shared Config Fields" [here](https://docs.aws.amazon.com/sdk-for-go/api/aws/session/#pkg-index).

Profiles can also be used with Terraform by setting `profile` in the `provider` block:

```hcl
provider "aws" {
  region = "us-west-2"
  profile = "pulumi-testing"
}
```

### Fallback: `assume-role`

If a tool, for whatever reason, doesn't support AWS profiles and insists on receiving credentials in environment variables, you can try [`assume-role`](https://github.com/remind101/assume-role).

## Reference `~/.aws/config`

This shared configuration file includes names and IAM roles for many of Pulumi's interesting AWS accounts.

```ini
[default]
region = us-west-2

[profile pulumi-testing]
role_arn = arn:aws:iam::086028354146:role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-testing-readonly]
role_arn = arn:aws:iam::086028354146:role/ReadOnly-OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-staging]
role_arn = arn:aws:iam::098437015098:role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-staging-readonly]
role_arn = arn:aws:iam::098437015098:role/ReadOnly-OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-production]
role_arn = arn:aws:iam::058607598222:role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-production-readonly]
role_arn = arn:aws:iam::058607598222:role/ReadOnly-OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-broomevideo]
role_arn = arn:aws:sts::034665324040:assumed-role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile ppc-production-learningmachine-prod]
role_arn = arn:aws:iam::396386917111:role/OrganizationAccountAccessRole
source_profile = default
region = us-east-1

[profile ppc-production-learningmachine-stage]
role_arn = arn:aws:iam::394069743421:role/OrganizationAccountAccessRole
source_profile = default
region = us-east-1

[profile ppc-production-learningmachine-malta]
role_arn = arn:aws:iam::894273677487:role/OrganizationAccountAccessRole
source_profile = default
region = eu-west-1

[profile ppc-production-learningmachine-auto]
role_arn = arn:aws:iam::750427423503:role/OrganizationAccountAccessRole
source_profile = default
region = us-east-1
```

## See also
[Links for assuming roles in the AWS console](Pulumi-AWS-accounts)