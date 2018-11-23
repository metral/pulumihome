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

### Analytics

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
* [[Working With DynamoDB]]
* [[Purging Soft-Deleted Stacks]]

#### Testing
##### Identities

We have tests that test the login flow and the relationships between pulumi `Organizations` and OAuth identity-backed organizations like GitHub's organizations (`GitHubOrganizations`) and GitLab's groups (`GitLabOrganizations`).

To that extent we have test accounts on GitHub and GitLab whose settings (username, org memberships etc.) must remain as-is. For details on what these test accounts are, see [this](https://docs.google.com/spreadsheets/d/1k-qy39wStLDdC9HfoPo3bdrk10jDASBMJvqeaP2zf_k/edit#gid=910719858) Google Doc on our Team Drive.

##### Stripe

Stripe provides test credit cards and corresponding tokens to use when programmatically creating charges. See [this](https://stripe.com/docs/testing) for more information.
