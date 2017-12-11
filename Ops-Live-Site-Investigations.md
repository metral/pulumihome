## Getting Site Administrator Access

The Pulumi Service has a way to grant a user access to view resources they normally wouldn't be authorized to. The `Users` table has a `site_access` field, which when non-zero, implies a special set of site access privileges.

For Pulumites who are investigating issues reported by customers, you may need to look at their organization, stacks, and resources as the way they do. So you'll need to set this `site_access` field on your account.

To access the instance database, you can use `scripts/launch-mysql-prompt.sh` in the `pulumi-service` repo. However, to have that use the right service instance, as opposed to a local dev stack you created, you must set the right `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `PULUMI_STACK_NAME_OVERRIDE` environment variables to correspond to the AWS account and stack name for the service instance you want to update.

```sql
# Query to grant user with GitHub login 'chrsmith' administrator access.
UPDATE Users SET site_access = 1 WHERE github_login = 'chrsmith'
```

Just remember as Uncle Ben said, with great power comes great responsibility. Also, this is something we'll eventually add more process and reporting around. e.g. only grant temporary access, and requiring another developer to approve it, etc.