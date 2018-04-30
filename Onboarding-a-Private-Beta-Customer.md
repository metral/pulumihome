## How to onboard a private beta customer

1. Create a welcome URL for the user, and put it in the [Pulumi Early Adopters spreadsheet](https://docs.google.com/spreadsheets/d/1JbFINleJ1-r4f-Q4m_ZrTdsZ7VOO7J-lznQamC7NEhE/edit#gid=0) (see notes below).
2. Send them an email invitation, making sure to cc support@pulumi.com. See example below.

After the invitation email has been sent:
- Ask Joe or Luke to add the user with as a collaborator on the [examples](https://github.com/pulumi/examples) repo. Also add them to all the core SDK ("to-be-open-source") repos. See GitHub notes below.
- Ask Joe, Luke or Donna to invite their email address as a single channel guest on the Slack channel #community-discussion.

Note: because our new onboarding process does not require a GitHub username, we will implement this feature: [Add a webhook to notify when a new user has signed up #1231](https://github.com/pulumi/pulumi-service/issues/1231). Before this feature has been rolled out, we will still ask folks for GitHub usernames, since the docs website has a number of links to the `examples` repo. GitHub usernames are stored in the `BetaAccess` table.

Note: as of 4/19, the login flow for docs.pulumi.com and the pulumi.com console have been unified. If a beta customer reports having trouble accessing docs.pulumi.com, they may need to do a logout/login on the console.

## What can private beta customers access?

Our engagement plan assumes that **each** private beta customer has access to the following:
- The docs website
- The pulumi.com service
- The GitHub [examples](https://github.com/pulumi/examples) repo, and ideally others
- (If they have accepted the invite) The Slack channel #community-discussion

## Generating an invite URL

As of 4/23, Donna, Joe, and Luke have access to generate codes.

To generate an invite URL, run [scripts/ops/generate-invite-codes.sh](https://github.com/pulumi/pulumi-service/blob/master/scripts/ops/generate-invite-codes.sh) in the Pulumi Service repo. 

## Slack #community-discussion Access

We add every user to the [#community-discussion](https://pulumi.slack.com/messages/C9SEFSC4C) channel, as a [Single Channel Guest](https://get.slack.help/hc/en-us/articles/202518103-Multi-Channel-and-Single-Channel-Guests). Currently only Slack administrators can invite single channel guests - please ping @joe, @luke or @Donna in #private-beta to ask that a new user be added.

## GitHub Read-Only Access

All beta users should have read-only access to the [pulumi/examples](https://github.com/pulumi/examples/settings/collaboration), since the documentation links there.

For *most* users, we will want to give them read-only access to **some** of our **other** GitHub repos.  We may elect not to do this if there is a concern over a specific person seeing our source code.  But in general, this will ensure a smoother experience, more akin to what will be available when we launch.  It lets people see source code and examples and, more importantly, read and file issues, all of which will help them be more productive using the beta.

The following are the repos we will typically give users **Collaborator** access to:

* [pulumi/pulumi](https://github.com/pulumi/pulumi/settings/collaboration)
* [pulumi/examples](https://github.com/pulumi/examples/settings/collaboration)
* [pulumi/docs](https://github.com/pulumi/docs/settings/collaboration)
* [pulumi/pulumi-cloud](https://github.com/pulumi/pulumi-cloud/settings/collaboration)
* [pulumi/pulumi-aws](https://github.com/pulumi/pulumi-aws/settings/collaboration)
* [pulumi/pulumi-azure](https://github.com/pulumi/pulumi-azure/settings/collaboration)
* [pulumi/pulumi-kubernetes](https://github.com/pulumi/pulumi-kubernetes/settings/collaboration)
* [pulumi/pulumi-terraform](https://github.com/pulumi/pulumi-terraform/settings/collaboration)

**Make sure collaborators have Read-only access:**

![image](https://user-images.githubusercontent.com/3953235/38164793-9021830c-34be-11e8-8e66-17bfd6d9d6b1.png)

In some cases, we may offer less -- for instance, if someone doesn't care about Azure, they may not care to see it -- however, in almost no circumstance should we go beyond this.  Specifically, the PPC and service repos, or anything pertaining to our SaaS product, are **never** shared with non-Pulumi employees.  We also don't share access to pulumi/home since it contains customer code and tests at the moment (this will be fixed before we open source).

## Personal Touch!

Please communicate all of the above with as personal a touch as you can.  This is a good way to drive engagement.  Please orient people towards [specific examples](https://github.com/pulumi/examples) that might be relevant to their work, and check in regularly, one on one, to see if we can be helpful in making sure they have a great experience.

## Example invitation email template

Make sure to CC support@pulumi.com so that other team members can see any issues or feedback.

See [Example invitation email](https://docs.google.com/document/d/11B5X_ghxlNUhSOz8q9OdL-RsKNyZEs5Fs9CRZsthTxs/edit#) for a formatted message that can be copied directly into email.

So that the invitee can send an invite to friends, include the survey link https://goo.gl/forms/NRVLdW2eLIKP9Bi23, which collects name, email, and currently GitHub username.