We are now archiving integration test timing and metadata for the following repos/tests:
* [pulumi/pulumi](https://github.com/pulumi/pulumi): `minimal`
* [pulumi/pulumi-cloud](https://github.com/pulumi/pulumi-cloud): `todo`, `crawler`, `countdown`, `containers`, `timers`, `httpEndpoint`
* [pulumi/home](https://github.com/pulumi/home): `learningmachine-cts`

Raw results are in the S3 bucket at https://s3.console.aws.amazon.com/s3/buckets/eng.pulumi.com/testreports/.

## Reports

We have an AWS Athena table and a few saved queries set up to provide reports over this data.  A few useful queries:

* [TestTimingsAllTime](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/ac1de368-a1c2-45d7-8d13-34f98db96c8c): Shows timing statistics  of each test phase across all tracked tests over all time.  

* [TestTimingsByDay](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/2cb12d55-c627-47b1-83ea-e088ec31c42a): Shows timing statistics of each test phase across all tracked tests broken down by day.  Useful for graphing trends over time.

* [TestTimingsByDay-LM](https://us-west-2.console.aws.amazon.com/athena/home?force&force=&region=us-west-2#query/saved/46ff66d7-9f38-49d2-bbd9-1f11dbcfaaa1): Shows timing statistics of each test phase across the Learning Machine test scenario, broken down by day.

* [TestTimingsByDayTravis-LM](https://us-west-2.console.aws.amazon.com/athena/home?force&force=&region=us-west-2#query/saved/38ff9b27-5d74-492e-a9df-de33e3824180): Shows timing statistics of each test phase across the Learning Machine test scenario, filtering to only runs done by Travis - broken down by day.

Example of data from these report:

| date       | testName     | stepName               | averageElapsedSeconds | medianElapsedSeconds | p10ElapsedSeconds | p90ElapsedSeconds | minElapsedSeconds | maxElapsedSeconds | count |
|------------|--------------|------------------------|-----------------------|----------------------|-------------------|-------------------|-------------------|-------------------|-------|
| 2017-11-14 | crawler      | pulumi-preview-initial | 44.64876842           | 52.58470917          | 35.87321091       | 54.26349258       | 35.87321091       | 54.26349258       | 4     |

An example summary of results for the `learningmachine-cts` stack from 11/14/2017-1/3/2018 is compiled at https://docs.google.com/document/d/1XXEeYWX8RlHKHILcsGVoEKk2DCZPiIMjIxPqW3ZhwzI.

## Dashboards

We do not yet have any dashboards set up to track this data over time.  Results can be downloaded from Athena and visualized manually in Excel/Sheets.  We expect to hook up PowerBI/Tableau/QuickSight or similar to this (and other) data in the future.  For now, please add links to any useful worksheets or reports here.

## Reporting for additional tests

Additional integration tests can report data into the same system using code like the following:

```golang
example := ex.With(integration.ProgramTestOptions{
    ReportStats: integration.NewS3Reporter("us-west-2", "eng.pulumi.com", "testreports"),
})
```