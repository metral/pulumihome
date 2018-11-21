# Querying Configuration Secret Usage

Queries to answer the following questions:

```
- # of config key/values in the system
- # of secrets in the system
- % of stacks that have config key/values
- % of stacks that have secrets
```

Start with all stacks that haven't been deleted.

```
MySQL [pulumi]> SELECT COUNT(*) FROM Programs WHERE deleted_at = 0;
+----------+
| COUNT(*) |
+----------+
|     4363 |
+----------+
1 row in set (0.21 sec)
```

Then join on `ProgramUpdates` on the latest version of the stack. This number is smaller to account for stacks that have been created but never updated. That is, rows where `programs.version = 0` which don't have a corresponding `ProgramUpdates` entry.

We also need to be sure to filter out `ProgramUpdates` rows where the `active` column is not 1. There may be multiple updates for a given stack version (e.g. previews), but only one will have the `active` field set to 1. (That was the update which actually made any resource changes and changed the stack's state.)

```
MySQL [pulumi]> SELECT COUNT(program_updates.id)
    -> FROM Programs AS programs
    -> JOIN ProgramUpdates AS program_updates
    ->     ON program_updates.program_id = programs.id
    ->     AND program_updates.version = programs.version
    ->     AND program_updates.active = 1
    -> WHERE programs.deleted_at = 0;
+----------------------------------+
| COUNT(program_updates.update_id) |
+----------------------------------+
|                             3666 |
+----------------------------------+
1 row in set (1.76 sec)
```

Configuration values are stored in the `ProgramUpdateMetadata` which contains configuration data (including secrets), as well as per-update environment data (e.g. the Git branch name).

Rows in the `ProgramUpdateMetadata` table are essentially a tuple of `(program_update_id, kind, name, value)`. The `kind` column is the type of metadata. The ones in particular we want are `config` and `config.secret`.

Breaking down all types of configuration types by value, across all active stacks is done via:

```
MySQL [pulumi]> SELECT COUNT(*), prog_updates_md.kind
    -> FROM Programs AS programs
    -> JOIN ProgramUpdates AS prog_updates
    ->     ON prog_updates.program_id = programs.id
    ->     AND prog_updates.version = programs.version
    ->     AND prog_updates.active = 1
    -> JOIN ProgramUpdateMetadata AS prog_updates_md ON prog_updates_md.program_update_id = prog_updates.id
    -> WHERE programs.deleted_at = 0
    -> GROUP BY prog_updates_md.kind;
+----------+---------------+
| COUNT(*) | kind          |
+----------+---------------+
|     6075 | config        |
|      647 | config.secret |
|     2865 | core          |
|     8392 | environment   |
+----------+---------------+
4 rows in set (1.08 sec)
```

Drilling in a bit further, we can then group by the stacks as well, to see which stacks have the largest number of secret configuration values in their latest update.

```
MySQL [pulumi]> SELECT COUNT(*), prog_updates_md.kind, programs.org_id, programs.stack_name
    -> FROM Programs AS programs
    -> JOIN ProgramUpdates AS prog_updates
    ->     ON prog_updates.program_id = programs.id
    ->     AND prog_updates.version = programs.version
    ->     AND prog_updates.active = 1
    -> JOIN ProgramUpdateMetadata AS prog_updates_md ON prog_updates_md.program_update_id = prog_updates.id
    -> WHERE
    ->     programs.deleted_at = 0
    ->     AND prog_updates_md.kind IN ("config.secret")
    -> GROUP BY prog_updates_md.kind, programs.id
    -> ORDER BY COUNT(*) DESC LIMIT 10;
+----------+---------------+--------------------------------------+--------------+
| COUNT(*) | kind          | org_id                               | stack_name   |
+----------+---------------+--------------------------------------+--------------+
|       18 | config.secret | 7cc62d65-a159-4df5-b826-7050fb95f92b | actiming-dev |
|       10 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | cts-malta    |
|       10 | config.secret | bbe948ea-9791-4f54-91d8-54869e77c5c4 | aws-stage    |
|       10 | config.secret | bbe948ea-9791-4f54-91d8-54869e77c5c4 | aws-prod     |
|       10 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | cts-cxc      |
|        9 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | cts-auto     |
|        9 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | cts-stage    |
|        9 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | govn-stage   |
|        9 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | cts-bahamas  |
|        9 | config.secret | d1ad898c-d600-4f21-9530-d5b107d1623f | govn-auto    |
+----------+---------------+--------------------------------------+--------------+
10 rows in set (0.98 sec)
```

Taking it a step further, we can look at the `ProgramUpdatesMetadata` `name` column to get an idea for what the secret configuration value is being used for.

```
MySQL [pulumi]> SELECT programs.stack_name, prog_updates_md.kind, prog_updates_md.name
    -> FROM Programs AS programs
    -> JOIN ProgramUpdates AS prog_updates
    ->     ON prog_updates.program_id = programs.id
    ->     AND prog_updates.version = programs.version
    ->     AND prog_updates.active = 1
    -> JOIN ProgramUpdateMetadata AS prog_updates_md ON prog_updates_md.program_update_id = prog_updates.id
    -> WHERE programs.deleted_at = 0 AND programs.stack_name = "mindbody-infra-gcp-alpha"
    -> AND prog_updates_md.kind IN ("config", "config.secret")
    -> ORDER BY prog_updates_md.kind;
+--------------------------+---------------+----------------------------------+
| stack_name               | kind          | name                             |
+--------------------------+---------------+----------------------------------+
| mindbody-infra-gcp-alpha | config        | gcp:project                      |
| mindbody-infra-gcp-alpha | config        | gcp:zone                         |
| mindbody-infra-gcp-alpha | config.secret | mindbody-infra:gclb-certificate  |
| mindbody-infra-gcp-alpha | config.secret | mindbody-infra:gclb-private-key  |
| mindbody-infra-gcp-alpha | config.secret | mindbody-infra:google-groups-api |
| mindbody-infra-gcp-alpha | config.secret | mindbody-infra:password          |
+--------------------------+---------------+----------------------------------+
6 rows in set (0.50 sec)
```