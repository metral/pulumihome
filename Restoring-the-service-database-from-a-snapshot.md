If something goes very wrong, we may need to replace the service Aurora cluster with a new one restored from a snapshot.

These steps were last tested against christian's dev instance on 2018-12-21. They take about one hour to complete.

## Choose a snapshot to restore

### Find the Aurora cluster

The ID for the active Aurora cluster can be found in the Pulumi snapshot or by running `pulumi stack`, after selecting the stack that you want to restore to (e.g. here, `testing`):

```
$ cd infrastructure
$ pulumi stack select testing
$ pulumi stack -i | grep databaseClusterId
    databaseClusterId          tf-20181221140607370200000001
```

Copy the cluster ID for use in the next step.

### Find the snapshot

[In the RDS console "Snapshots" view](https://us-west-2.console.aws.amazon.com/rds/home?region=us-west-2#db-snapshots:), filter the list of snapshots by the `databaseClusterId` you obtained above and select the "best" available snapshot -- hopefully the most recent, unless we're restoring an old DB because of longer-term data corruption. (You can sort by the Snapshot Creation Time column.)

Copy the snapshot name for use below. It will look like `rds:tf-20180503000923005800000002-2018-05-11-11-599`.

## Add new Aurora cluster to infrastructure

Edit the `infrastructure` Pulumi program (specifically `database.ts`) to:

1. Add a new Aurora cluster created from the identified snapshot.
1. Add an Aurora instance targeting the new cluster.
1. Export the new instance to the rest of the infrastructure.

This process will not remove the old resources as part of this deployment -- that adds delay and unnecessary risk to recovery. Also note that the new resources (cluster, instance and s3 bucket) will be given new names to prevent Pulumi from failing with a `Duplicate resource URN` error.

Using [this commit](https://github.com/pulumi/pulumi-service/commit/584213163bbb11d8b53d390ce728e5c4da1138d0) as a template, change the value of `databaseCluster2`'s `snapshotIdentifier` to the snapshot name you identified above:

```
let databaseCluster2 = new aws.rds.Cluster("databaseCluster2", {
    ...
    snapshotIdentifier: "YOUR_SNAPSHOT_NAME",
});
```

## Preview your changes

```
$ cd infrastructure
$ make build && pulumi preview
```

You should see that a new cluster, instance snapshot will be created:

```
Previewing update (testing):

     Type                        Name                       Plan        Info
     pulumi:pulumi:Stack         pulumi-service-testing 
 +   ├─ aws:s3:Bucket            snapshot2                  create
 +   ├─ aws:rds:Cluster          databaseCluster2           create
 +   ├─ aws:rds:ClusterInstance  databaseInstance2          create
 +-  ├─ aws:ecs:TaskDefinition   apiTaskDefinition          replace     [diff: ~containerDefinitions]
 ~   └─ aws:ecs:Service          api-                       update      [diff: ~taskDefinition]

Resources:
    + 3 to create
    ~ 1 to update
    +-1 to replace
    3 changes. 48 unchanged
```

## Deploy

Submit the changes through Travis, or if you have access keys and an AWS profile set up locally for the environment you're targeting, you can run `./scripts/update-stack.sh` directly by setting the appropriate values of `AWS_PROFILE` and `PULUMI_STACK_NAME_OVERRIDE`:

```
$ AWS_PROFILE=pulumi-testing PULUMI_STACK_NAME_OVERRIDE=testing ./scripts/update-stack.sh
```

Creating the new Aurora cluster will take about 40 minutes, and the instance pointing to it will be ready in another 10 minutes or so. Do not commit your changes to GitHub -- we'll need to make a few more changes first.

## Removing the inactive cluster

At the end of this process we'll have two running DB clusters. When you're ready to dispose of the inactive one, you'll need to dispose of the cluster and instance first, followed by the cluster's s3 bucket, to allow AWS to save a final snapshot of the cluster into that bucket. If you attempt to destroy all three resources at once, `pulumi update` will likely fail on the absence of the final-snapshot bucket. 

See this issue for additional context: https://github.com/pulumi/pulumi-service/issues/2610 
