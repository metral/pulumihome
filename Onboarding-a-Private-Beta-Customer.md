There are three steps to onboarding a Private Beta customer.

## Pulumi Console

We maintain a whitelist for users who have access to the Pulumi service.  It is stored in a database.

This isn't exposed via the REST API, so to add new users we'll need to manually add them to the database via SQL-query. Assuming you have `launch-mysql-prompt.sh` setup ([notes on wiki](https://github.com/pulumi/home/wiki/Putting-PPCs-in-maintenance-mode#database-access)), the query to add a user is:

```
# Add GitHub user "lukehoban" to the service whitelist.
INSERT INTO `BetaAccess` (`kind`, `name`) VALUES ("gh-user", "lukehoban");
```

## Pulumi Docs Website

## Slack #community-discussion Access

We add every user to the [#community-discussion](https://pulumi.slack.com/messages/C9SEFSC4C) channel, as a [Single Channel Guest](https://get.slack.help/hc/en-us/articles/202518103-Multi-Channel-and-Single-Channel-Guests).

## (Optional) GitHub Read-Only Access

## Personal Touch!

Please communicate all of the above with as personal a touch as you can.  This is a good way to drive engagement.  Please orient people towards [specific examples](https://github.com/pulumi/examples) that might be relevant to their work, and check in regularly, one on one, to see if we can be helpful in making sure they have a great experience.