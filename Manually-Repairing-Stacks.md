At this point, we have minimal tooling to help with repairing stacks that are "live" inside of a PPC.  Below are some common activities in addition to the work item tracking making each activity easier to perform.

# Manually Repairing a Checkpoint

If a stack's checkpoint is wrong -- for example, due to drift with the target environment -- or needs to be manually updated, say, due to a bug in Pulumi, this is currently a manual process.

:warning: ***Note**: first thing's first: it is imperative that you don't perform these operations while in-flight activity on a given stack is happening, otherwise unpredictable results such as stack corruption may occur.*

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
