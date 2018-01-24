# Accessing PPC logs for a given update

If you need to see the output that `pulumi` printed for a given update or preview in the cloud (say to understand if a failure is due to a buggy program or there's some underlying Pulumi issue), you can use the `ppc` tool to fetch these, without needing access to the customer's organization.

## Things You Need:

1. The Update ID that failed (the notification in Slack should have this)
2. The URI and token for the PPC endpoint. The notification in Slack tells you
   the PPC that the update was in and the the "Pulumi Private Clouds"
   spreedsheet in Google Drive > PulumiAllEmployees > Engineering > Keys And
   Passwords has the endpoint and token.
3. A build of `ppc` which has the ability to tail log files. Building 
   `pulumi/pulumi-ppc` from HEAD will give you this.

## Process

### Figure out if an Apply or Preview failed.

The slack notification may tell you this, but if it doesn't, do the following:
   
```
# ppc --cloud <URL-of-cloud-endpoint> --token <token> update <update-id>
```
   
This will produce a bunch of output. You are interested in the "State" field. If
it is `apply failed`, `applying` or `apply succceeded`, this is an apply,
otherwise it is a preview.

### Use --tail argument to ppc update or apply to see the logs

Use PPC to get the logs, depending on if it is a preview or apply, the command is slightly different
   
For a preview:

```
# ppc --cloud <URL-of-cloud-endpoint> --token <token> update preview --tail <update-id>
```
   
For an apply:

```
# ppc --cloud <URL-of-cloud-endpoint> --token <token> update apply --tail <update-id>
```
  
Note in the above the only difference is if you pass the `preview` or `apply`
argument after `update`.
  
Information about the update (including all of the logs) will be printed to the
console.
