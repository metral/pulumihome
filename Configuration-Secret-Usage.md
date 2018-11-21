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

```
MySQL [pulumi]> SELECT COUNT(program_updates.id)
    -> FROM Programs AS programs
    -> JOIN ProgramUpdates AS program_updates ON program_updates.program_id = programs.id AND program_updates.version = programs.version
    -> WHERE programs.deleted_at = 0;
+----------------------------------+
| COUNT(program_updates.update_id) |
+----------------------------------+
|                             3666 |
+----------------------------------+
1 row in set (1.76 sec)
```

Configuration values are stored in the `ProgramUpdateMetadata` which contains configuration data (including secrets), as well as per-update environment data (e.g. the Git branch name).

Rows in the `ProgramUpdateMetadata` table are essentially a tuple of `(program_update_id, kind, name, value)`. The `kind` column is the type of metadata. The most current set of possible values are `core`, `environment`, `config`, or `config.secret`.
