The ability to "attach" a PPC to an organization is not exposed through the UI of the Pulumi Service. Instead, you'll either need to issue the requests using `cURL` or create the records manually via the database.

The following commands show how to attach a PPC to an organization using the REST API. Note however that only administrators of an organization (i.e. admins of the GitHub organization) will be able to do this.

```
# Pulumi organization to attach the PPC to.
export USER_TO_REGISTER=moolumi
# Name of the PPC (should match spreadsheet of doom)
export PPC_NAME="dogfood-ppc-staging"

# Pulumi access token for the user making the API calls.
# That user needs to be an admin in the GitHub org backing ${USER_TO_REGISTER}.
export PULUMI_ACCESS_TOKEN="..."
# API endpoint of the service instance to add the PPC to.
export PULUMI_API="https://api.pulumi.com"

# PPC contact information, from spreadsheet of doom.
export PPC_ACCESS_TOKEN="xxx"
export PPC_ENDPOINT="https://xxx.execute-api.us-west-2.amazonaws.com/stage/"

# Attach the cloud to the given organization.
# Flags (2) means opt-out of health checks. (Maybe not what you want.)
curl \
    -i \
    -H "Authorization: $PULUMI_ACCESS_TOKEN" \
    -X POST \
    -d"{\"name\": \"$PPC_NAME\", \"endpoint\": \"$PPC_ENDPOINT\", \"accessToken\": \"$PPC_ACCESS_TOKEN\", \"flags\": 2}" \
    "$PULUMI_API/orgs/$USER_TO_REGISTER/clouds"

# Optionally mark that new PPC as the "default" for that organization.
# (This feature and capability will hopefully go away in M12.)
curl \
    -i \
    -H "Authorization: $PULUMI_ACCESS_TOKEN" \
    -X POST \
    "$PULUMI_API/orgs/$USER_TO_REGISTER/clouds/$PPC_NAME/default"

# A PPC can optionally have configuration policy to override Pulumi program
# configuration passed to it. The following configuration defines a default
# value for "aws:region" if not set, and requires "aws:accessKey" be present.
export PPC_CONFIG='
{
  "configuration": [
    { "key": "aws:accessKey", "type": "require", "value": ""},
    { "key": "aws:secretKey", "type": "require", "value": ""},
    { "key": "aws:region", "type": "default", "value": "us-west-2" }
  ]
}'

curl \
    -H "Authorization: token $PULUMI_ACCESS_TOKEN" \
    -X PATCH \
    -d "$PPC_CONFIG" \
    "$PULUMI_API/orgs/$USER_TO_REGISTER/clouds/$PPC_NAME"
```