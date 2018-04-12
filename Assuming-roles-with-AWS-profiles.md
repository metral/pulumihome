The AWS SDKs and CLI let you switch between different named bundles of configuration called _profiles_. You can read more about using profiles in the CLI [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html).

> Using profiles with the AWS SDK for Go may require setting `AWS_SDK_LOAD_CONFIG`. See "Shared Config Fields" [here](https://docs.aws.amazon.com/sdk-for-go/api/aws/session/#pkg-index).

Profiles let you use _assumed-role_ credentials easily. For example, if your `~/.aws/config` file has your user credentials:

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
region = us-west-2
```

> Each profile has to specify a region. One might expect `region` or other settings to be inherited, but `source_profile` is only for specifying credentials to assume the IAM role.

Now you can use that profile with the CLI:

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
$ AWS_PROFILE=pulumi-testing aws sts get-caller-identity
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

Profiles can also be used with Terraform by setting `profile` in the `provider` block:

```hcl
provider "aws" {
  region = "us-west-2"
  profile = "pulumi-testing"
}
```

## Reference `~/.aws/config`

This shared configuration file includes names and IAM roles for many of Pulumi's interesting AWS accounts.

```ini
[default]
region = us-west-2

[profile pulumi-testing]
role_arn = arn:aws:iam::086028354146:role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-staging]
role_arn = arn:aws:iam::098437015098:role/OrganizationAccountAccessRole
source_profile = default
region = us-west-2

[profile pulumi-production]
role_arn = arn:aws:iam::058607598222:role/OrganizationAccountAccessRole
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