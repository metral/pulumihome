Each service stack has an Aurora instance that's only available from within the service VPC. You can create an SSH tunnel to connect with the MySQL client.

## Prerequisites
1. Make sure the `mysql` command line tool is installed on your machine.
2. Download [`pulumi.058607598222.us-west-2.pem`](https://drive.google.com/open?id=0B_ivBLhaCF_ceHIxRWpsM1NSbzg), where `058607598222` is the [[ID of the production account|Pulumi AWS accounts]].
3. Copy the file to your `~/.ssh` folder.
4. `chmod 400 ~/.ssh/pulumi.058607598222.us-west-2.pem`
5. Make sure your AWS shared configuration file includes a `pulumi-testing` (or `staging`, or `production`) profile, as described [[here|Assuming roles with AWS profiles#reference-awsconfig]].

## Launching a MySQL prompt

Run the following in the root of the `pulumi-service` repository:
```bash
export AWS_PROFILE=pulumi-testing # or "staging" or "production"
export PULUMI_STACK_NAME_OVERRIDE=testing # or "staging" or "production"
export AWS_DEFAULT_REGION=us-west-2 # This may be required, depending on how your profile is configured

pulumi logout
PULUMI_DEBUG_COMMANDS=1 pulumi login --local
./scripts/launch-mysql-prompt.sh
unset AWS_PROFILE
unset PULUMI_STACK_NAME_OVERRIDE
```

If you receive an error such as `fatal error: An error occurred (404) when calling the HeadObject operation: Key "v1/production.json" does not exist`, you are most likely in the wrong account. Ensure you're using the right profile.

If you are prompted for your Pulumi Access token, that means you didn't correctly set the mode to local stacks (aka "fire and forget"). Run `pulumi logout` and `pulumi login --local` again.