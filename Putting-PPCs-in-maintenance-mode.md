PPCs can be put into "maintenance mode", which will return 503 "Service Unavailable" for any request to create stacks, update stacks, etc.

There is no way to do this through the UI. It must be done by directly editing the database. Each PPC (or row in the `Clouds` table) has a `flags` column that contains bit flags for various status.

The maintenance mode flag is the 0x1 bit.

### Database Access

#### Prerequisites
1. Make sure the `mysql` command line tool is installed on your machine
2. Download [pulumi.058607598222.us-west-2.pem](https://drive.google.com/open?id=0B_ivBLhaCF_ceHIxRWpsM1NSbzg)
3. Copy the file to your `~/.ssh` folder
4. `chmod 400 ~/.ssh/pulumi.058607598222.us-west-2.pem`
5. Make sure your AWS shared configuration file includes a `pulumi-testing` (or `staging`, or `production`) profile, as described [[here|Assuming roles with AWS profiles#reference-awsconfig]].

#### Launch a mysql prompt

1. Make

1. Run the following in the root of the `pulumi-service` repository:
    ```bash
    export AWS_PROFILE=pulumi-testing # or "staging" or "production"
    export PULUMI_STACK_NAME_OVERRIDE=testing # or "staging" or "production"

    pulumi logout
    pulumi login --cloud-url local://
    ./scripts/launch-mysql-prompt.sh
    unset AWS_PROFILE
    unset PULUMI_STACK_NAME_OVERRIDE
    ```

If you receive an error such as `fatal error: An error occurred (404) when calling the HeadObject operation: Key "v1/production.json" does not exist`, you are most likely in the wrong account. Ensure you're using the right profile.

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