Welcome to the Pulumi Team Wiki!

Please take a look around and please make it better as you see fit.

## General

* [[Effective Slack]]
* [[Onboarding a Private Beta Customer]]
* [[Hackathon Ideas]]

## Engineering

### Operations

* [[On-Call]]
* [[Enterprise and Team Support]]
* Our AWS accounts
    * [Google Doc](https://docs.google.com/document/d/1Do4YHOQSM6yxnXVef0dcsZ_8sqpOLm4w6Tri0KfzUFM)
    * [[List with "assume role" links|Pulumi AWS accounts]]
    * [[Assuming roles with AWS profiles]] (includes reference `~/.aws/config`)
* Deployments
    * [Weekly release process](https://github.com/pulumi/home/wiki/Weekly-release-process)
    * [Updating the Service](Updating-the-Service) 
    * [Updating PPCs](Updating-PPCs)
    * [Building the SDK](https://github.com/pulumi/home/wiki/Producing-an-SDK)
    * [[Updating the Docs Website]]
* [Live Site Investigations](Ops-Live-Site-Investigations)
  * [Production Incident Reports](Production-Incident-Reports)
* [[Pulumi Service Stacks]]
* [[Pulumi Websites]]
* [[Manually-Repairing-Pulumi-Cloud-Stacks]]
* How our Release Bucket Permissions Work ([here](https://github.com/pulumi/home/issues/57#issuecomment-344809733) and [here](https://github.com/pulumi/home/issues/64#issuecomment-349088546))
* [Reading List](Ops-Reading-List)
* [[Manually setting up billing for an Organization]]

### Administration

* [[Transferring Stacks]] (beta)

### Analytics

- [[Metabase]]
- [Analytics queries on prod database](https://github.com/pulumi/home/wiki/Analytics-queries-on-prod)
- [[Configuration Secret Usage]]

### Daily Development

* [[New engineering team hire]]
* Setting up a development machine
    - [[Development on macOS]]
    - [[Development on Linux]]
    - [[Development on Windows]]
* [[Code Review at Pulumi]]
* [[Planning, Work Items, and Changelog]]
* [[Managing Repo Versions]]
* [Running the Pulumi Service](https://github.com/pulumi/home/wiki/Running-the-Pulumi-Service)
* [Integration Test Performance](https://github.com/pulumi/home/wiki/Integration-Test-Performance-Reports)
* [Pulumi Design Notes (PDNs)](https://drive.google.com/drive/folders/0B0siYR6Ttr5LVk85eU9NYmI1UW8)
* [[Purging Soft-Deleted Stacks]]

#### Testing

> Google Drive folder with information about test data is [here](https://drive.google.com/drive/u/1/folders/1SkLYWgEQpqkI7sqnWkuiBx8_u_ivMB6x).

##### Identities

We have tests that test the login flow and the relationships between pulumi `Organizations` and OAuth identity-backed organizations like GitHub's organizations (`GitHubOrganizations`), GitLab's groups (`GitLabOrganizations`), and Bitbucket Teams (`BitbucketOrganizations`).

To that extent we have test accounts on GitHub, GitLab, and Bitbucket whose settings (username, org memberships etc.) must remain as-is. For details on what these test accounts are, see [this](https://docs.google.com/spreadsheets/d/1k-qy39wStLDdC9HfoPo3bdrk10jDASBMJvqeaP2zf_k/edit#gid=910719858) Google Doc on our Team Drive.

##### Atlassian Bitbucket Tests

Bitbucket access tokens expire in 2 hours. The mechanism to get a fresh access token is to use the `refresh_token` grant type. The refresh token for a user is tied to a specific set of OAuth client credentials. This means, one cannot take the refresh token for a user produced by our staging OAuth consumer, and try to use it with our testing OAuth consumer. They are not interchangeable.

We use AWS Secrets Manager to store the refresh tokens for the test accounts and these can be used only with our testing OAuth consumer with the `refresh_token` OAuth grant type. Furthermore, Travis CI doesn't have env vars per branch, so we have the same refresh tokens in AWS Secrets Manager in all of the AWS accounts. When you run the tests locally, you need to have the client credentials for our testing OAuth consumer under the env vars [`TEST_BITBUCKET_OAUTH_ID`](https://github.com/pulumi/pulumi-service/blob/master/cmd/service/model/bitbucket/testutils.go#L54) and [`TEST_BITBUCKET_OAUTH_SECRET`](https://github.com/pulumi/pulumi-service/blob/master/cmd/service/model/bitbucket/testutils.go#L55). Otherwise, the tests where an Atlassian user is involved, will be skipped.

The testing environment's OAuth consumer can be found [here](https://bitbucket.org/account/user/pulumi/api). It's the **Pulumi (Testing)** app. If you don't have access to the **pulumi** Bitbucket Team, contact one of the admins (@praneetloke or @chrsmith).

##### Stripe

Stripe provides test credit cards and corresponding tokens to use when pro grammatically creating charges. See [this](https://stripe.com/docs/testing) for more information.
