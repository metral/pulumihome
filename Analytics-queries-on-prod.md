Queries for collecting usage data from production systems - the production database and the production cloudfront logs.

# Prod Database

A list of useful queries on how customers are using the service. Connect to the prod database by following the instructions in [[Accessing the service database in production]].



## Users and deployments

```sql
select Users.github_login, count(*) as updates 
from Users 
   inner join ProgramUpdates on Users.id = ProgramUpdates.requested_by 
group by Users.id 
order by updates desc;
```

## Users who have logged in to the service

```sql
select github_login, email, created from Users order by created desc;
```

## Deployments per user 

```sql
select Users.email, Users.created as OnboardDate, count(ProgramUpdates.id) as deployments
from Users
   left join ProgramUpdates on Users.id = ProgramUpdates.requested_by
group by Users.id
order by Users.created;
```

## Stacks per organization (stacks that have not been deleted)

```sql
select o.github_login, count(p.stack_name) as Stacks
from Programs p
   left join Organizations o on p.org_id = o.id
group by o.github_login
order by Stacks;
```

## Deployments and last deployment time per user

```sql
select u.email, count(p.id) as Deployments, max(p.created) as LastDeploymentTime
from Users u
   left join ProgramUpdates p on u.id = p.requested_by
group by u.id
order by u.created;
```

## Used invite codes

```sql
select create_user.github_login as Inviter, invite_user.github_login as InvitedGitHub, invite_user.email InvitedEmail, invite_user.created as JoinDate
from InviteCodes i
   inner join Users create_user on create_user.id = i.created_by
   inner join Users invite_user on invite_user.id = i.used_by
order by JoinDate;
```

## Beta access users who have logged in
Users who were given direct access to the private beta

```sql
select b.name, b.modified from BetaAccess b
   inner join Users u on u.github_login = b.name
order by b.modified;
```

# Prod Cloudfront Logs

We have an Athena table defined in our prod account in `us-west-2`.

## Downloads by date and route

See [in console](https://us-west-2.console.aws.amazon.com/athena/home?force&force=&region=us-west-2#query/saved/5e5b129a-1f6f-4981-8a66-63b475b65681).

```sql
SELECT date, uri, count(*) AS downloads
FROM cloudfront_logs
GROUP BY  date, uri
ORDER BY  date, uri;
```