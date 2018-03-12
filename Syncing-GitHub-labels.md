## About the tool

To sync GitHub labels across repos, you can use the the script [apply-labels.sh](https://github.com/pulumi/home/blob/master/scripts/github-labels/apply-labels.sh). See usage instructions in the script itself.

## How to use
- When you add a repo, add it to the [`REPOS` array](https://github.com/pulumi/home/blob/master/scripts/github-labels/apply-labels.sh#L33). Run the script with `--dry-run` first in order to make sure everything behaves as expected.
- When you want to add a label, add its information to [labels.json](https://github.com/pulumi/home/blob/master/scripts/github-labels/labels.json). Run the script with `--dry-run` to see what changes will be made.
- To change a label description name, you can use the `aliases` array in `labels.json`. However, the colors of the labels must match, due to a limitation in the underlying tool (see [Merging labels doesn't seem to work #12](https://github.com/Financial-Times/github-label-sync/issues/12)). To work around this, run the tool twice: 
  - Change the label name and add the old name to the `aliases` array, and run the tool.
  - Change the label color and run the tool again.
