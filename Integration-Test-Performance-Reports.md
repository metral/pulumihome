We are now archiving integration test timing and metadata for the following repos/tests:
* `pulumi/pulumi`: `minimal`
* `pulumi/pulumi`: All (`todo`, `crawler`, `countdown`, `containers`, `timers`, `httpEndpoint`)

Raw results are in the S3 bucket at https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/testreports/.

Additional integration tests can report data into the same system using code like the following:

```golang
example := ex.With(integration.ProgramTestOptions{
    ReportStats: integration.NewS3Reporter("us-west-2", "eng.pulumi.com", "testreports"),
})
```

We have an AWS Athena table and a few saved queries set up to provide reports over this data.  Two useful queries:

[TestTimingsAllTime](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/ac1de368-a1c2-45d7-8d13-34f98db96c8c): Shows timing statistics  of each test phase across all tracked tests over all time.  

[TestTimingsByDay](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/2cb12d55-c627-47b1-83ea-e088ec31c42a): Shows timing statistics of each test phase across all tracked tests broken down by day.  Useful for graphing trends over time.


