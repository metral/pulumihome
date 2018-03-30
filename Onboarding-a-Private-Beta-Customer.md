There are three steps to onboarding a Private Beta customer.

* Make sure they have an NPM token assigned in the [Pulumi Early Adopters spreadsheet](https://docs.google.com/spreadsheets/d/1JbFINleJ1-r4f-Q4m_ZrTdsZ7VOO7J-lznQamC7NEhE/edit#gid=0) (can be the same token as someone else)
* Add their GitHub username to the docs.pulumi.com access list by following the instructions below
* Ask Joe or Luke to invite their email address as a single channel guest on the Slack channel #community-discussion.
* [Optional for now - but soon required] Add their GitHub username to the Pulumi console whitelist
* [Optional] If they need GitHub access to one or more projects, we will add them as Collaborators on a case-by-case basis.
* Send them an invitation email!

## Pulumi Docs Website

See https://github.com/pulumi/pulumi-service/blob/master/cmd/docs/DEVOPS.md#adding-a-github-user-to-the-whitelist.

## Pulumi Console

We maintain a whitelist for users who have access to the Pulumi service.  It is stored in a database.

This isn't exposed via the REST API, so to add new users we'll need to manually add them to the database via SQL-query. Assuming you have `launch-mysql-prompt.sh` setup ([notes on wiki](https://github.com/pulumi/home/wiki/Putting-PPCs-in-maintenance-mode#database-access)), the query to add a user is:

```
# Add GitHub user "lukehoban" to the service whitelist.
INSERT INTO `BetaAccess` (`kind`, `name`) VALUES ("gh-user", "lukehoban");
```

## Slack #community-discussion Access

We add every user to the [#community-discussion](https://pulumi.slack.com/messages/C9SEFSC4C) channel, as a [Single Channel Guest](https://get.slack.help/hc/en-us/articles/202518103-Multi-Channel-and-Single-Channel-Guests).  Currently only Slack administrators can invite single channel guests - please ping @joe or @luke in #private-beta to ask that a new user be added.

## (Optional) GitHub Read-Only Access

We will on a case-by-case basis add read-only Collaborators.  If you believe someone should have GitHub access, please post a message on #private-beta to discuss.

## Personal Touch!

Please communicate all of the above with as personal a touch as you can.  This is a good way to drive engagement.  Please orient people towards [specific examples](https://github.com/pulumi/examples) that might be relevant to their work, and check in regularly, one on one, to see if we can be helpful in making sure they have a great experience.

## Example invitation email template

Hi <Name>,

Thank you for agreeing to participate in the Pulumi private beta! I’m excited for you to test drive Pulumi and share your feedback.

Pulumi is a new programming platform for the cloud, which merges together cloud infrastructure management, application deployment, and serverless containers and functions into a unified development experience. I think you’re going to love it. :)

To make sure we have time to integrate your feedback into the product, we’re eager to hear from you early (and often!).  If you’re open to it, please choose a 20 min slot on my calendar for a call, ideally within the next week or so. Otherwise, I’ll follow up with you next week to see how things are going.

I’m adding you to pulumi.slack.com so that you can easily chat with me or my team. But feel free to reach out through any other channel as well.  We’re interested in all feedback, ranging from nits on the docs, to the CLI flow, to feedback on the overall experience.

Access instructions
Go to https://docs.pulumi.com and sign in with your GitHub account
In the instructions for configuring your NPM client, use the token XXXXX
(Optional) View GitHub repos in the Pulumi organization (look for an organization invite).
 
A small reminder: this is a private beta, so please don’t share product details with others. But, if you have a friend or colleague who might be interested in providing feedback, please let us know!

I’m looking forward to your feedback!

Donna
