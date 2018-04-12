PPCs can be put into "maintenance mode", which will return 503 "Service Unavailable" for any request to create stacks, update stacks, etc.

There is no way to do this through the UI. It must be done by directly editing the database. Each PPC (or row in the `Clouds` table) has a `flags` column that contains bit flags for various status.

The maintenance mode flag is the 0x1 bit.

### Database Access

#### Prerequisites
1. Make sure the `mysql` command line tool is installed on your machine
1. Download [pulumi.058607598222.us-west-2.pem](https://drive.google.com/open?id=0B_ivBLhaCF_ceHIxRWpsM1NSbzg)
1. Copy the file to your `~/.ssh` folder
1. `chmod 400 ~/.ssh/pulumi.058607598222.us-west-2.pem`

#### Launch a mysql prompt

1. Open [AWS Role Account Passwords](https://docs.google.com/document/d/1p1jvOxbyiy_2OYdtWMjgY3W7vVZXx3Hq3O8Eq1Euea8/edit#) to get the `AWS_SECRET_KEY_ID` and `AWS_SECRET_ACCESS_KEY` values for your environment. For example, user **travis-cicd** in the **production** account.

1. Run the following: 
    ```bash
    export AWS_SECRET_KEY_ID=access_key_from_doc
    export AWS_SECRET_ACCESS_KEY=secret_key_from_doc
    export PULUMI_STACK_NAME_OVERRIDE=production # or "staging" or "testing"

    pulumi logout
    pulumi login --cloud-url local://
    ./scripts/launch-mysql-prompt.sh # run in [pulumi-service] repo
    ```

If you receive an error such as `fatal error: An error occurred (404) when calling the HeadObject operation: Key "v1/production.json" does not exist`, you are most likely in the wrong account.  Ensure that you are using the correct credentials using environment variables from the above instructions.

If you are prompted for your Pulumi Access token, that means you didn't correctly set the mode to local stacks (aka "fire and forget"). Run `pulumi logout` and `pulumi login --cloud-url local://` again.

### Setting Maintenance Mode

```
# List the Clouds you are interested in updating. Note the current flag values.
# the FAKE-PPC-FOR-TESTING has 0x2 set (opt out of health checks).
SELECT name, id, org_id, flags FROM Clouds;
+----------------------+--------------------------------------+--------------------------------------+-------+
| name                 | id                                   | org_id                               | flags |
+----------------------+--------------------------------------+--------------------------------------+-------+
| FAKE-PPC-FOR-TESTING | 2632f283-c2f5-434f-a8ac-1e8bb54c8deb | feacc792-460f-4525-a091-e8de1f6ef34c |     2 |
...
```

```
# Set all PPCs associated with a given Organization to be in maintenance mode.
# Use the org_id value you saw above.
UPDATE Clouds SET flags = 1 WHERE org_id = "feacc792-460f-4525-a091-e8de1f6ef34c";
```

### Clearing Maintenance Mode

To take a PPC out of maintenance mode, just remove the 0x1 bit from the `flags` column. The 99.9% case is just setting it to be 0. But if the PPC had some special situation (e.g. other `flags` set) be sure to restore them as needed.

```
# Clear the "maintenance mode" flag for PPCs.
UPDATE Clouds SET flags = 0 WHERE org_id = "feacc792-460f-4525-a091-e8de1f6ef34c";
```