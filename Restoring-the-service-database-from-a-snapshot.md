If something goes very wrong, we may need to replace the service Aurora cluster with a new one restored from a snapshot.

These steps were last tested against mattdr's dev instance on 2018-05-11. They take about two hours.

## Choose a snapshot to restore

### Find the Aurora cluster

The Aurora cluster can be found in the Pulumi snapshot or from `pulumi stack -i | grep databaseClusterId`.

It can also be found if you know the database _instance_ identifier, which you can find from e.g. the `PULUMI_DATABASE_ENDPOINT` environment variable in the definition for the service ECS task. Go to the RDS console in the appropriate account and region, find the instance in the "Instances" view, and look for the "Cluster" section of the instance details page.

The cluster and instance IDs will look like `tf-20171108003110516600000002`.

### Find the snapshot

In the RDS console "Snapshots" view, filter the snapshots by the cluster ID and select the "best" available snapshot -- hopefully the most recent, unless we're restoring an old DB because of longer-term data corruption.

The snapshot name will look like `rds:tf-20180503000923005800000002-2018-05-11-11-599`.

## Add new Aurora cluster to infrastructure

Edit the `infrastructure` Pulumi program (specifically `database.ts`) to:
1. Add a new Aurora cluster created from the identified snapshot.
2. Add an Aurora instance targeting the new cluster.
3. Export the new instance to the rest of the infrastructure.

Don't remove the old resources as part of this deployment -- that adds delay and unnecessary risk to recovery. New resources will need new names or Pulumi will fail with a `Duplicate resource URN` error.

Use [this commit](https://github.com/pulumi/pulumi-service/commit/584213163bbb11d8b53d390ce728e5c4da1138d0) as a template.

## Deploy

Submit the changes through Travis or run `./scripts/update-stack.sh` locally with the appropriate values of `AWS_PROFILE`, `PULUMI_STACK_NAME_OVERRIDE`.

> Until [pulumi/pulumi-terraform#151](/pulumi/pulumi-terraform/issues/151) is fixed, the cluster may fail to create due to a timeout.
>
> As a crude workaround, build your own `pulumi-aws` after manually changing every timeout in `vendor/terraform-providers/terraform-provider-aws/aws/resource_aws_rds_cluster.go` from `d.Timeout(...)` to `120 * time.Minute`.

Creating the new Aurora cluster will take about 40 minutes, and the instance pointing to it will be ready in another 10 minutes or so.