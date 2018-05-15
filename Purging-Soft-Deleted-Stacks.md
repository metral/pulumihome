# Purging Soft-deleted Stack Data

The Pulumi Service performs a soft-delete for stacks, allowing users to undelete that stack and
recover update history, checkpoints, etc. This is done by setting a `deleted_at` timestamp on the
`Programs` row in our database. The value `0` meaning the `Program` (stack) has not been deleted.

In order to purge soft-deleted stacks from the database we generally want to delete all `Programs`
rows where the `deleted_at` column is greater than `0`. However, a couple more steps are required.

The `Programs` table is how the service views a stack. (And its corresponding `ProgramUpdates` and
`Resources` tables). However, hosted stacks are also persisted in the database as well as have
their checkpoints stored on S3.

While deleting the `Programs` row will also wipe out its `ProgramUpdates` and `Resources` via
cascading deletes, we also need to delete the lower-level `Stacks` rows too. Deleting `Stacks`
will in-turn delete `Updates`, `UpdateCheckpointInfo`, `UpdateLogs`, and other data via
cascading delete as well.

## Step 1. Confirm the stacks to delete

Double check the set of stacks that are soft deleted and need to be cleaned up.

This query shows the total count of soft-deleted stacks by organization:

```sql
MySQL [pulumi]> SELECT o.name, count(*) FROM Programs AS p
JOIN Organizations AS o ON p.org_id = o.id
WHERE p.deleted_at > 0 GROUP BY o.name;
+----------------+----------+
| name           | count(*) |
+----------------+----------+
| Moolumi        |    93138 |
| Pulumi         |     1729 |
| Sean Gillespie |        4 |
+----------------+----------+
```

Assuming there are no soft-deleted stacks to keep around in the Pulumi or Moolumi organizations,
we can identify the set of `Programs` to delete with:

```sql
SELECT p.stack_name
FROM Programs AS p
JOIN Organizations AS o ON p.org_id = o.ID
WHERE o.github_login IN ("moolumi", "pulumi") AND p.deleted_at > 0
LIMIT 10;
```

Finally, get a baseline by running:

```sql
SELECT TABLE_NAME, TABLE_ROWS FROM information_schema.tables WHERE table_schema = "pulumi";
```

## Step 2. ~~Back up Checkpoint files~~

Since we store checkpoint files on S3, we should keep that list around to delete them at some
later time.

However, since we have versioning enabled for the S3 bucket storing checkpoints, deleting the
checkpoint files won't actually do anything. We'd need to wipe out the history for those items
as well.

For the time being we are just ignoring that and leaving checkpoint files on S3.

```sql
SELECT COUNT(update_checkpoint_info.data_key)
FROM UpdateCheckpointInfo AS update_checkpoint_info
JOIN Updates AS updates ON update_checkpoint_info.update_id = updates.id
JOIN Stacks AS stacks ON updates.stack_id = stacks.id
WHERE stacks.id IN (
    SELECT stack_id FROM Programs AS p
    JOIN Organizations AS o ON p.org_id = o.ID
    WHERE o.github_login IN ("moolumi", "pulumi") AND p.deleted_at > 0
);
```

To actually back them up. However, to do this from your local machine you need
to invoke the command via your client. (After running `create-db-tunnel.sh`.)

```bash
mysql -u readonly -e 'SELECT update_checkpoint_info.data_key
FROM UpdateCheckpointInfo AS update_checkpoint_info
JOIN Updates AS updates ON update_checkpoint_info.update_id = updates.id
JOIN Stacks AS stacks ON updates.stack_id = stacks.id
WHERE stacks.id IN (
    SELECT stack_id FROM Programs AS p
    JOIN Organizations AS o ON p.org_id = o.ID
    WHERE o.github_login IN ("moolumi", "pulumi") AND p.deleted_at > 0
)
' --batch --host=localhost --protocol=TCP pulumi > data-key-backups.txt
```

## Step 2. Delete the Stacks, Updates, etc.

We have a circular FK relationship, so we first bulk update the `ActiveUpdateID` for every
`Stacks` row we want to delete to be `NULL`.

Note that depending on the query used, the operation can take 10s of seconds to several hours.

First mark the `ActiveUpdateID` `NULL` for the `Stacks` we want to delete.

```sql
UPDATE Stacks, Programs, Organizations
SET Stacks.active_update_id = NULL
WHERE
    Stacks.id = Programs.stack_id AND
    Programs.deleted_at > 0 AND
    Programs.org_id = Organizations.id AND
    Organizations.github_login IN ("moolumi", "pulumi");
```

Next, delete the `Stacks` which will cascade into deleting data in other tables too. The main
query to run is like the following, however it is too slow and will likely cause problems.

```sql
# DELETE Stacks
# FROM Stacks, Programs, Organizations
# WHERE
#     Stacks.id = Programs.stack_id AND
#     Programs.deleted_at > 0 AND
#     Programs.org_id = Organizations.id AND
#     Organizations.github_login in ("pulumi", "moolumi");
```

Instead, shard the delete operation based on the `Stacks.id` column. Where the first letter
is `[0-9a-f]`.

```sql
DELETE Stacks
FROM Stacks, Programs, Organizations
WHERE
    Stacks.id = Programs.stack_id AND
    Programs.deleted_at > 0 AND
    Programs.org_id = Organizations.id AND
    Organizations.github_login in ("pulumi", "moolumi") AND
    Stacks.id LIKE "f%";
```

After running the 16 delete queries, we have now deleted all of the "low level" `Stacks` rows.

## 3. Delete the Programs, ProgramUpdates, etc.

We delete `Programs` in the same way as deleting `Stacks`, sharding by the first character
of the `id` column.

Like before, run the query 16 times replacing the `LIKE` prefix.

```sql
DELETE Programs
FROM Programs, Organizations
WHERE
    Programs.deleted_at > 0 AND
    Programs.org_id = Organizations.id AND
    Organizations.github_login in ("pulumi", "moolumi") AND
    Programs.id LIKE "0%";
```

The final step is to get the updated `TABLE_ROWS` information to ensure that everything
was removed:

```sql
SELECT TABLE_NAME, TABLE_ROWS FROM information_schema.tables WHERE table_schema = "pulumi";
```

## Troubleshooting / Diangostics

Long query that appears to be stuck? Have another engineer SSH into the database and run
`SHOW FULL PROCESSLIST;`. That will provide some diagnostic information for active queries.

Feel free to abort any that are problematic via, `KILL XXX;` (Where `XXX` is the query ID.)

To see the aggregate row count across each table, run:

```sql
SELECT TABLE_NAME, TABLE_ROWS FROM information_schema.tables WHERE table_schema = "pulumi";
```