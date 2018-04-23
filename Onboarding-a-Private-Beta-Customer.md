## How to onboard a private beta customer

1. Make sure they have an NPM token assigned in the [Pulumi Early Adopters spreadsheet](https://docs.google.com/spreadsheets/d/1JbFINleJ1-r4f-Q4m_ZrTdsZ7VOO7J-lznQamC7NEhE/edit#gid=0). The convention is that for each "wave" of new users, we use the same NPM token. (This is because the tokens can only be revoked by us, using the Pulumi Bot npm user.)
1. Add their GitHub username to the docs.pulumi.com access list, following the instructions below
1. Add their GitHub username to the Pulumi console whitelist, following the instructions below
1. Send them an email invitation, making sure to cc support@pulumi.com. See example below.

After the invitation email has been sent, ask Joe or Luke to do the following:
- Add the user with as a collaborator on the [examples](https://github.com/pulumi/examples) repo. Once there is sufficient cleanup of other repos, add them to all the core SDK ("to-be-open-source") repos. See GitHub notes below.
- Ask Joe or Luke to invite their email address as a single channel guest on the Slack channel #community-discussion.

## What can private beta customers access?

Our engagement plan assumes that **each** private beta customer has access to the following:
- The docs website
- The pulumi.com service
- The GitHub [examples](https://github.com/pulumi/examples) repo, and ideally others
- (If they have accepted the invite) The Slack channel #community-discussion

## Pulumi Docs Website

For users whose organization and email domain are not in the whitelist (i.e, private beta customers), their GitHub username can be added to a whitelist. The process below provides access only to [docs.pulumi.com](https://docs.pulumi.com), not the pulumi.com service.

To add a user to the whitelist, do the following:

- SSH into the docs website VM using the [instructions here](https://github.com/pulumi/home/wiki/Updating-the-Docs-Website)
- Open `/home/ubuntu/pulumi-docs/.env` in an editor
- Add their GitHub username to the list `DOCS_GITHUB_LOGINS`:

  ```
  # Pulumi friends and family.
  DOCS_GITHUB_LOGINS=lumi-test-3,malayeri
  ```

After doing this, you'll need to restart the website service for changes to take effect:

```
$ sudo service pulumi-docs restart
```

## Pulumi Console

We maintain a whitelist for users who have access to the Pulumi service, stored in a database. However, since the access list isn't exposed via the REST API, you need to add new users manually to the database with a SQL query.

1. Set up `launch-mysql-prompt.sh`, following the steps in [Database Access](https://github.com/pulumi/home/wiki/Putting-PPCs-in-maintenance-mode#database-access). Use the user **travis-cicd** in the **production** account.

1. Add a user with the following query:

   ```
   # Add GitHub user "lukehoban" to the service whitelist.
   INSERT INTO `BetaAccess` (`kind`, `name`) VALUES ("gh-user", "lukehoban");
   ```

## Slack #community-discussion Access

We add every user to the [#community-discussion](https://pulumi.slack.com/messages/C9SEFSC4C) channel, as a [Single Channel Guest](https://get.slack.help/hc/en-us/articles/202518103-Multi-Channel-and-Single-Channel-Guests).  Currently only Slack administrators can invite single channel guests - please ping @joe or @luke in #private-beta to ask that a new user be added.

## GitHub Read-Only Access

Although not a strict requirement, for *most* users, we will want to give them read-only access to **some** of our GitHub repos.  We may elect not to do this if there is a concern over a specific person seeing our source code.  But in general, this will ensure a smoother experience, more akin to what will be available when we launch.  It lets people see source code and examples and, more importantly, read and file issues, all of which will help them be more productive using the beta.

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