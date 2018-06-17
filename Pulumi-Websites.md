In addition to the Pulumi Service, there is a small constellation of websites for various things. This page calls out their general workings.

## pulumi.io

Is the documentation website.

It is managed by a Pulumi program in `/infrastructure`. The stacks are:

- `pulumi/pulumi-io-production`
- `pulumi/pulumi-io-staging`

The AWS resources for both stacks are in the production AWS account.

Travis is configured such that pushes to `master` will update the staging stack and updates to the `production` branch will update the production stack.

## www.pulumi.com

Is the company homepage

It is managed by a Pulumi program in `/infrastructure`. The stacks are:

- `pulumi/www.pulumi.com-production`
- `pulumi/www.pulumi.com-staging`

The AWS resources for both stacks are in the production AWS account.

Travis is configured such that pushes to `master` will update the staging stack and updates to the `production` branch will update the production stack.

## blog.pulumi.com

Details unknown. Ask Marc.

## docs.pulumi.com

https://docs.pulumi.com was the old website we used for hosting Pulumi documentation. As of June, 2018 it was replaced with https://pulumi.io.

The website is powered by a combination of S3 bucket and CloudFront distribution that serve a 301 redirect to https://pulumi.io. The resources were manually provisioned and reside in the production AWS account.

## slack.pulumi.com

https://slack.pulumi.com is a simple app that automates the process of getting invited to a Slack community. See [home #286](https://github.com/pulumi/home/issues/286) for context.

It was stood up manually due to time constraints, and is cobbled together using:

- An S3 bucket (slack.pulumi.com)
- A CloudFront distribution to serve the content
- A API Gateway for serving the REST API (with a single endpoint)
- A Lambda for calling the Slack API to make the request
