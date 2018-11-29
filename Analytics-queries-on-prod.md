Queries for collecting usage data from production systems - the production database and the production cloudfront logs.

# Prod Database

A list of useful queries on how customers are using the service. Connect to the prod database by following the instructions in [[Accessing the service database in production]].

### Users and deployments

```sql
select Users.github_login, count(*) as updates 
from Users 
   inner join ProgramUpdates on Users.id = ProgramUpdates.requested_by 
group by Users.id 
order by updates desc;
```

### Users who have logged in to the service

```sql
select github_login, email, created from Users order by created desc;
```

### Deployments and last deployment time per user

**Filter out users who haven't deployed**

```sql
select * from (
    select u.email, count(p.id) as Deployments, max(p.created) as LastDeploymentTime
    from Users u
    left join ProgramUpdates p on u.id = p.requested_by
    group by u.id
    order by LastDeploymentTime desc 
) as t
where t.Deployments > 0;
```

**Users filtered by sign-up date**

```sql
select * from (
    select u.email, u.created, count(p.id) as Deployments, max(p.created) as LastDeploymentTime
    from Users u
    left join ProgramUpdates p on u.id = p.requested_by
    group by u.id
    order by LastDeploymentTime desc 
) as t
where t.Deployments > 0 and t.created > date("2018-06-18 00:00:00");
```

**All users**

```sql
select u.email, count(p.id) as Deployments, max(p.created) as LastDeploymentTime
from Users u
   left join ProgramUpdates p on u.id = p.requested_by
group by u.id
order by LastDeploymentTime desc;
```

### Deployments per user 

```sql
select Users.email, Users.github_login, Users.created as OnboardDate, count(ProgramUpdates.id) as deployments
from Users
   left join ProgramUpdates on Users.id = ProgramUpdates.requested_by
group by Users.id
order by Users.created;
```

GitHub username instead of email:

```sql
select u.github_login, count(p.id) as Deployments, max(p.created) as LastDeploymentTime
from Users u
   left join ProgramUpdates p on u.id = p.requested_by
group by u.id
order by LastDeploymentTime desc;
```

### Stacks per organization (stacks that have not been deleted)

```sql
select o.github_login, count(p.stack_name) as Stacks
from Programs p
   left join Organizations o on p.org_id = o.id
group by o.github_login
order by Stacks;
```

# Event Data

The Pulumi Service has an `Events` table we write one-time event information to for our own analytics purposes. Longer term this should go in another database, or some 3rd party service. But for the time being its a simple table with schema `created, user_id, event, [optional] resource_id`.

NOTE: These queries will need to be updated every time we add a new event type (integer value).

## Events over the past day

```
SELECT
    CASE event
    WHEN  1 THEN "NewUserEvent"
    WHEN  2 THEN "NewStackEvent"
    WHEN  3 THEN "StackUpdateEvent"
    WHEN  4 THEN "LoginEvent"
    WHEN  5 THEN "UpdateAccessTokenEvent"
    WHEN  6 THEN "StackDeletedEvent"
    WHEN  7 THEN "StackPreviewEvent"
    WHEN  8 THEN "StackRefreshEvent"
    WHEN  9 THEN "StackDestroyEvent"
    WHEN 10 THEN "StackImportEvent"
    WHEN 11 THEN "StackExportEvent"
    WHEN 12 THEN "NewOrganizationEvent"
    WHEN 13 THEN "InviteStackCollaborator"
    WHEN 14 THEN "InviteOrganizationMemberEvent"
    END as event_type, COUNT(*)
FROM Events
WHERE created <= CURDATE() && created > (CURDATE() - INTERVAL 1 DAY)
GROUP BY event;
```

# Prod Cloudfront Logs

We have an Athena table defined in our prod account in `us-west-2`.

### Downloads by date and route

See [in console](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/5e5b129a-1f6f-4981-8a66-63b475b65681) in the production account (058607598222)
run the following queries. (All against the "default" database, which already has the
expected tables.)

```sql
SELECT date, uri, count(*) AS downloads
FROM cloudfront_logs
GROUP BY  date, uri
ORDER BY  date, uri;
```

To exclude users from Seattle (the best proxy currently for non-Pulumi users):

```sql
SELECT date,
       uri,
       count(*) AS downloads
FROM cloudfront_logs
WHERE location NOT LIKE 'SEA%'
GROUP BY  date, uri
ORDER BY  date, uri;
```

To get all unique IP addresses that have downloaded Pulumi along with the first time they downloaded:

```sql
SELECT requestip, min(date) as firstSeen, count(*) as numDownloads
FROM cloudfront_logs
WHERE location NOT LIKE 'SEA%'
GROUP BY requestip
ORDER BY firstSeen
```

Look at long-tail requests over the past couple of days:

```sql
SELECT date, host, uri, time, timetaken
FROM cloudfront_logs
WHERE date BETWEEN date '2018-06-10' AND date '2018-06-13'
ORDER BY timetaken desc
LIMIT 100;
```

Or from the ALB:

```sql
SELECT domain_name, request_processing_time, target_processing_time, response_processing_time, time, request_verb, request_url
FROM alb_logs
WHERE time BETWEEN '2018-06-10T00:00:00Z' and '2018-06-14T00:00:00Z'
ORDER BY target_processing_time DESC
LIMIT 100;
```

# Prod https://pulumi.io CloudFront Logs

All searches on docs site:

```
SELECT date, time, location, requestip, querystring, referrer FROM pulumiio_cloudfront_logs
WHERE uri LIKE '%search.html%'
ORDER BY date DESC, time DESC
```

# Prod https://api.pulumi.io ALB Logs

See [in console](https://us-west-2.console.aws.amazon.com/athena/home?force&region=us-west-2#query/saved/5e5b129a-1f6f-4981-8a66-63b475b65681) in the production account (058607598222)
you can use Athena to query the ALB logs.

CLI versions (via `User-Agent` header) over the past 30 days:

```sql
SELECT DISTINCT user_agent FROM "default"."alb_logs" 
WHERE from_iso8601_timestamp(time) > (current_timestamp - interval '30' day) AND regexp_like(user_agent, 'pulumi-cli.*\(v[0-9]+\.[0-9]+\.[0-9]+;.*')
```