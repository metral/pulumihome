PPCs can be put into "maintenance mode", which will return 503 "Service Unavailable" for any request to create stacks, update stacks, etc.

There is no way to do this through the UI. It must be done by directly editing the database. Each PPC (or row in the `Clouds` table) has a `flags` column that contains bit flags for various status.

The maintenance mode flag is the 0x1 bit.

### Database Access

See [[accessing the service database in production]].

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