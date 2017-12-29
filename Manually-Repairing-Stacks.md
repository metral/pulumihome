At this point, we have minimal tooling to help with repairing stacks that are "live" inside of a PPC.  Below are some common activities in addition to the work item tracking making each activity easier to perform.

# Manually Repairing a Checkpoint

If a stack's checkpoint is wrong -- for example, due to drift with the target environment -- or needs to be manually updated, say, due to a bug in Pulumi, this is currently a manual process.

The first step is to get into the AWS account for the target PPC, in the usual ways.  Select the correct region.

Then, go to the DynamoDB Tables.  There are three per PPC: stacks (the one we want), updates (a history of stack updates), and results (detailed log entries for all past updates).  Pick the stacks table.

In here, you will see an entry for each of the stacks in this PPC.  This example has three:

![image](https://user-images.githubusercontent.com/3953235/34442651-5cf98b2a-ec78-11e7-947f-a768346a234b.png)

Unfortunately, there are UUIDs rather than textual names that you are probably expecting.  To find the stack name for a given entry, take the `ActiveUpdate` value and correlate it back with an entry in the updates table:

For instance, take that first row:

![image](https://user-images.githubusercontent.com/3953235/34442712-c530f322-ec78-11e7-9521-d4412b24293d.png)

And then go to the entries table and look for `27a4723a-51d9-4edd-bd43-24ee31f9ea4f`:

![image](https://user-images.githubusercontent.com/3953235/34442735-e36b9270-ec78-11e7-97c4-7078f203e129.png)

We can see that this is the `cts-malta` stack.

From there, we can now go to the S3 bucket for this PPC, and find the associated checkpoint files.

See [pulumi/pulumi-ppc#43](https://github.com/pulumi/pulumi-ppc/issues/43) for a work item tracking adding tools to make this easier.

# PPC Update Failed Partway Through

If the PPC update fails partway through, it will leave the `Invalid` property set to `true`.  As a result, any subsequent attempts to preview or update this stack will yield the error `checkpoint is in an unknown state`.

To fix this, manually inspect and repair the checkpoint file per the above process.  Then, and only then, is it safe to unset the `Invalid` bit in the DynamoDB table for this stack:

![image](https://user-images.githubusercontent.com/3953235/34442577-e63852e6-ec77-11e7-8a23-873e3ea81b10.png)
