## Getting Site Administrator Access

The Pulumi Service has a way to grant a user access to view resources they normally wouldn't be authorized to. The `Users` table has a `site_access` field, which when non-zero, implies a special set of site access privileges.

For Pulumites who are investigating issues reported by customers, you may need to look at their organization, stacks, and resources as the way they do. So you'll need to set this `site_access` field on your account.

TODO: Write steps for how to access the production database.

```sql
# Query to grant user with GitHub login 'chrsmith' administrator access.
UPDATE Users SET site_access = 1 WHERE github_login = 'chrsmith'
```

Just remember as Uncle Ben said, with great power comes great responsibility. Also, this is something we'll eventually add more process and reporting around. e.g. only grant temporary access, and requiring another developer to approve it, etc.