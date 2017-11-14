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

## Reports

We have an AWS Athena table and a few saved queries set up to provide reports over this data.  Two useful queries:

[TestTimingsAllTime](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/ac1de368-a1c2-45d7-8d13-34f98db96c8c): Shows timing statistics  of each test phase across all tracked tests over all time.  

[TestTimingsByDay](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/2cb12d55-c627-47b1-83ea-e088ec31c42a): Shows timing statistics of each test phase across all tracked tests broken down by day.  Useful for graphing trends over time.

Example of data from this report:

| date       | testName     | stepName               | averageElapsedSeconds | medianElapsedSeconds | p10ElapsedSeconds | p90ElapsedSeconds | minElapsedSeconds | maxElapsedSeconds | count |
|------------|--------------|------------------------|-----------------------|----------------------|-------------------|-------------------|-------------------|-------------------|-------|
| 2017-11-14 | crawler      | pulumi-preview-initial | 44.64876842           | 52.58470917          | 35.87321091       | 54.26349258       | 35.87321091       | 54.26349258       | 4     |
| 2017-11-14 | containers   | pulumi-preview-initial | 46.20082951           | 51.51049423          | 36.7494812        | 52.2133522        | 36.7494812        | 52.2133522        | 4     |
| 2017-11-14 | countdown    | pulumi-update-initial  | 53.98117447           | 55.47883224          | 47.28860474       | 59.07168198       | 47.28860474       | 59.07168198       | 4     |
| 2017-11-14 | timers       | pulumi-update-initial  | 75.60348129           | 78.62023163          | 70.83625031       | 80.89278412       | 70.83625031       | 80.89278412       | 4     |
| 2017-11-14 | todo         | pulumi-update-initial  | 97.99446487           | 104.2246475          | 81.93669891       | 121.76297         | 81.93669891       | 121.76297         | 4     |
| 2017-11-14 | crawler      | pulumi-update-initial  | 141.8597056           | 128.8568573          | 91.72515869       | 204.9971008       | 91.72515869       | 204.9971008       | 3     |
| 2017-11-14 | containers   | pulumi-destroy         | 807.135376            | 830.5335083          | 783.7372437       | 830.5335083       | 783.7372437       | 830.5335083       | 2     |
| 2017-11-14 | containers   | pulumi-update-initial  | 1935.139771           | 1980.150879          | 1890.128662       | 1980.150879       | 1890.128662       | 1980.150879       | 2     |
