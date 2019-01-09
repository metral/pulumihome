Pulumi Site administrators have the ability to transfer stacks between organizations. This is something we need to do from time to time for our potential customers, especially during the onboarding process.

> **IMPORTANT** There are some situations where transferring ownership will cause problems. For example, the KMS key used to encrypt stack configuration is different. So you won't be able to decrypt config secrets for older stack updates, etc.

## Ensure you have Site Admin Permission

The Stack transfer feature requires that the `site_access` field of your user account has the `SiteAdministrator` bit set. There is no way to do this from the UI, and must be performed manually via poking the database.

```sql
# Query some users to get site access status
MySQL [pulumi]> SELECT github_login, site_access FROM Users WHERE github_login IN ("chrsmith", "lukehoban", "clstokes");
+--------------+-------------+
| github_login | site_access |
+--------------+-------------+
| chrsmith     |           1 |
| clstokes     |           0 |
| lukehoban    |           1 |
+--------------+-------------+
3 rows in set (0.02 sec)

# Set the site_access bit to 1.
MySQL [pulumi]> UPDATE Users SET site_access = 1 WHERE github_login = "clstokes";
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

## Transfer Stack

The feature was added in [pulumi-service 2568](https://github.com/pulumi/pulumi-service/pull/2568), but to use the feature run the following `cURL` command:

```bash
export PULUMI_API=https://api.pulumi.com
export STACK_NAME=exciting-stack-name
export FROM_ORG=old-and-busted-org
export TO_ORG=exciting-new-org

cat > ./payload.json <<EOF
{
    "stackName": "${STACK_NAME}",
    "fromOrg": "${FROM_ORG}",
    "toOrg": "${TO_ORG}"
} 
EOF

curl \
    -X POST \
    -H "Authorization: token ${PULUMI_ACCESS_TOKEN}" \
    -d @payload.json \
    ${PULUMI_API}/api/admin/stacks/move
```

