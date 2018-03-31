There are three steps to onboarding a Private Beta customer.

* Make sure they have an NPM token assigned in the [Pulumi Early Adopters spreadsheet](https://docs.google.com/spreadsheets/d/1JbFINleJ1-r4f-Q4m_ZrTdsZ7VOO7J-lznQamC7NEhE/edit#gid=0) (can be the same token as someone else)
* Add their GitHub username to the docs.pulumi.com access list by following the instructions below
* Ask Joe or Luke to invite their email address as a single channel guest on the Slack channel #community-discussion.
* Add their GitHub username to the Pulumi console whitelist
* (Optional) Add GitHub access as collaborators to the core SDK ("to-be-open-source") repos
* Send them an invitation email!

## Pulumi Docs Website

For users whose organization and email domain are not in the whitelist (i.e, private beta customers), their GitHub username can be added to a whitelist. The process below provides access only to [docs.pulumi.com](https://docs.pulumi.com), not beta.pulumi.com.

To add a user to the whitelist, do the following:
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

We maintain a whitelist for users who have access to the Pulumi service.  It is stored in a database.

This isn't exposed via the REST API, so to add new users we'll need to manually add them to the database via SQL-query. Assuming you have `launch-mysql-prompt.sh` setup ([notes on wiki](https://github.com/pulumi/home/wiki/Putting-PPCs-in-maintenance-mode#database-access)), the query to add a user is:

```
# Add GitHub user "lukehoban" to the service whitelist.
INSERT INTO `BetaAccess` (`kind`, `name`) VALUES ("gh-user", "lukehoban");
```

## Slack #community-discussion Access

We add every user to the [#community-discussion](https://pulumi.slack.com/messages/C9SEFSC4C) channel, as a [Single Channel Guest](https://get.slack.help/hc/en-us/articles/202518103-Multi-Channel-and-Single-Channel-Guests).  Currently only Slack administrators can invite single channel guests - please ping @joe or @luke in #private-beta to ask that a new user be added.

## GitHub Read-Only Access

Although not a strict requirement, for *most* users, we will want to give them read-only access to **some** of our GitHub repos.  We may elect not to do this if there is a concern over a specific person seeing our source code.  But in general, this will ensure a smoother experience, more akin to what will be available when we launch.  It lets people see source code and examples and, more importantly, read and file issues, all of which will help them be more productive using the beta.

The following are the repos we will typically add access to:

* [pulumi/pulumi](https://github.com/pulumi/pulumi/settings/collaboration)
* [pulumi/examples](https://github.com/pulumi/examples/settings/collaboration)
* [pulumi/docs](https://github.com/pulumi/docs/settings/collaboration)
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

See [Example invitation email](https://docs.google.com/document/d/11B5X_ghxlNUhSOz8q9OdL-RsKNyZEs5Fs9CRZsthTxs/edit#) for formatted message that can be copied directly into email.

Hi `<Name>`,

Thank you for agreeing to participate in the Pulumi private beta! I’m excited for you to test drive Pulumi and share your feedback.

Pulumi is a new programming platform for the cloud, which merges together cloud infrastructure management, application deployment, and serverless containers and functions into a unified development experience. I think you’re going to love it. :)

To make sure we have time to integrate your feedback into the product, we’re eager to hear from you early (and often!).  If you’re open to it, please choose a 20 min slot on my calendar for a call, ideally within the next week or so. Otherwise, I’ll follow up with you next week to see how things are going.

I’m adding you to pulumi.slack.com so that you can easily chat with me or my team. But feel free to reach out through any other channel as well.  We’re interested in all feedback, ranging from nits on the docs, to the CLI flow, to feedback on the overall experience.

Access instructions
- Go to https://docs.pulumi.com and sign in with your GitHub account
- In the instructions for configuring your NPM client, use the token XXXXX
- (Optional) View GitHub repos in the Pulumi organization (look for an organization invite).
 
A small reminder: this is a private beta, so please don’t share product details with others. But, if you have a friend or colleague who might be interested in providing feedback, please let us know!

I’m looking forward to your feedback!

Donna
