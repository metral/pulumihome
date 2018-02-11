At this point, we have a couple paths forward if a Pulumi Cloud stack needs to be repaired. In a broad sense, all avenues are similar: the stack's latest checkpoint needs to be fetched, repaired, and re-uploaded to the relevant PPC. The particulars, however, differ vastly between approaches. The approach you take will depend on the access you have and the particulars of the situation; they are listed below in order from "most access/simplest" to "least access/most complex".

Note that the first step in each approach is to obtain the logs for the update that left the checkpoint in an inconsistent state. Once this has been done, you should examine the logs in order to determine what operation (if any) was in progress when the update failure occurred. It is very possible that the operation succeeded but the checkpoint failed to serialize: if this is the case, you must update the checkpoint to reflect the changes or manually roll the changes back s.t. the checkpoint and the resource state are in sync. If the operation failed, however, then no changes are necessary to the checkpoint file.

# Repairing a Stack Using the `pulumi` CLI

If you have access to the organization associated with the stack that needs repair, you can use the `pulumi` CLI and the Pulumi Console to perform the necessary operations.
1. Fetch the logs for the failed update from the Pulumi Console. This will involve navigating to the stack's update history page and examining each update's logs until the failed update is found.
2. Examine the logs as described above.
3. Export the stack's checkpoint: `pulumi stack export > deployment.current.json`
4. Copy the exported deployment to a new file: `cp deployment.{current,repaired}.json`
5. Edit the copied deployment as described above.
6. Diff the repaired deployment against its baseline for completeness: `diff deployment.{current,repaired}.json`
7. Import the repaired deployment: `pulumi stack import < deployment.repaired.json`

At this point the stack should now be in a consistent state.

# Repairing a Stack Using the `ppc` CLI

If you do not have access to the organization associated with the stack that needs repair, you can use the `ppc` CLI to perform the necessary operations, though you will need some additional information in order to do so.
1. Determine which PPC is responsible for the stack that needs repair.
2. Look up that PPC's endpoint and token from our [handy spreadsheet](https://docs.google.com/spreadsheets/d/1ASpyMHUvC1rCN_6cRP6tq1D3378YzSC0PlHzvv_G42I/edit#gid=0)
3. Determine the GUID of the stack that needs repair. This may come from any number of locations (e.g. a Slack notification), but at a last resort can be pulled from Pulumi.com's database tables. (_TODO: we could use a service CLI/admin page/something to perform this lookup_)
4. Determine the failed update by examining each recent update in the stack's history. The stack's history can be fetch by running `ppc --cloud [PPC endpoint, including trailing slash] --token [PPC token] stack updates [stack GUID]`
5. Fetch the logs for the failed update: `ppc [cloud and token args] update logs --apply [update GUID]`
6. Examine the logs as described above.
7. Export the stack's checkpoint: `ppc [cloud and token args] stack [stack GUID] --export deployment.current.json`
8. Copy the exported deployment to a new file: `cp deployment.{current,repaired}.json`
9. Edit the copied deployment as described above.
10. Diff the repaired deployment against its baseline for completeness: `diff deployment.{current,repaired}.json`
11. Create an update that will import the repaired deployment: `ppc [cloud and token args] update create --import deployment.repaired.json`. This will display the GUID of the created update on the console if all is well.
12. Apply the import: `ppc [cloud and token args] update apply [update GUID from step 11]`. This should print `update succeeded` if all is well.

# Repairing a Stack Using the AWS Console

:warning: ***Note**: first things first: it is imperative that you don't perform these operations while in-flight activity on a given stack is happening, otherwise unpredictable results such as stack corruption may occur.*

The first step is to get into the AWS account for the target PPC, in the usual ways.  Select the correct region.

Then, go to the DynamoDB Tables.  There are three per PPC: stacks (the one we want), updates (a history of stack updates), and results (detailed log entries for all past updates).  Pick the stacks table.

In here, you will see an entry for each of the stacks in this PPC.  This example has three:

![image](https://user-images.githubusercontent.com/3953235/34442651-5cf98b2a-ec78-11e7-947f-a768346a234b.png)

Unfortunately, there are UUIDs rather than textual names that you are probably expecting.  To find the stack name for a given entry, take the `ActiveUpdate` value and correlate it back with an entry in the updates table:

For instance, take that first row:

![image](https://user-images.githubusercontent.com/3953235/34442712-c530f322-ec78-11e7-9521-d4412b24293d.png)

And then go to the updates table and look for `27a4723a-51d9-4edd-bd43-24ee31f9ea4f`:

![image](https://user-images.githubusercontent.com/3953235/34442735-e36b9270-ec78-11e7-97c4-7078f203e129.png)

We can see that this is the `cts-malta` stack.

From there, we can now go to the S3 bucket for this PPC, and find the associated checkpoint files.  First, go back to the stacks table, and expand the `Checkpoint` property to reveal its S3 path:

![image](https://user-images.githubusercontent.com/3953235/34442798-7a48891e-ec79-11e7-9036-1d69bcbbdc4f.png)

Now, navigate to that path in the S3 service console:

![image](https://user-images.githubusercontent.com/3953235/34442822-9fffaf84-ec79-11e7-898a-c998e05195c7.png)

Notice that the file is at the top when sorted by Last Modified descending, but you may also use the search window to locate the object. From there, you may Download the file, edit it, and re-Upload it in place of the old one.  The old version will be saved automatically in case you need to restore it.  From there, any PPC operations will use the new edited checkpoint.

Instead of using the console, you may also use the command line.  For example:

```bash
$ aws --profile the-ppc-profile s3 cp s3://learningmachine-ppc-malta-bucket97748f67/checkpoints/64bf3496-9c01-4eb2-999d-0d31ceace5ef/36d6ec7ae9e24adadda405df5b560e150fd8d4298797acf17c02052727667c5f ~/temp/36d6ec7ae9e24adadda405df5b560e150fd8d4298797acf17c02052727667c5f
# edit the file ...
$ aws --profile the-ppc-profile s3 cp ~/temp/36d6ec7ae9e24adadda405df5b560e150fd8d4298797acf17c02052727667c5f s3://learningmachine-ppc-malta-bucket97748f67/checkpoints/64bf3496-9c01-4eb2-999d-0d31ceace5ef/36d6ec7ae9e24adadda405df5b560e150fd8d4298797acf17c02052727667c5f
```

:construction: See [pulumi/pulumi-ppc#43](https://github.com/pulumi/pulumi-ppc/issues/43) for a work item tracking adding tools to make this easier.

# PPC Update Failed Partway Through

If the PPC update fails partway through, it will leave the `Invalid` property set to `true`.  As a result, any subsequent attempts to preview or update this stack will yield the error `checkpoint is in an unknown state`.

To fix this, manually inspect and repair the checkpoint file per the above process.  Then, and only then, is it safe to unset the `Invalid` bit in the DynamoDB table for this stack (a.k.a., its `DataKey`):

![image](https://user-images.githubusercontent.com/3953235/34442577-e63852e6-ec77-11e7-8a23-873e3ea81b10.png)