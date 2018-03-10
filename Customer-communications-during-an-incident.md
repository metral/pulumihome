Keeping our customers in the loop is a critical part of responding to outages. When our service goes down and customers can’t get their work done, they need to know we value their time and we’re working to fix the problem.

# Guidelines

## Who
Everyone at Pulumi is trusted to "speak for the company". Use these guidelines and your best judgment to help make outages less confusing or painful for our customers.

The oncall responder is responsible for making sure customers are informed. They can do it themselves or delegate.

## Where
For now our primary mode of communication with customers is shared Slack channels. This will evolve over time to include a status page, social media, e-mail, etc.

## When and what
When we recognize our service is having a problem, we should reach out to customers quickly to let them know we’re investigating. Ideally we can share something about the shape of the problem, like “elevated error rates” or “failed updates”, but if we don’t have those details after a few minutes that shouldn’t stop us from communicating.

During an outage, update customers at least once an hour. In each message, describe what we’re doing to solve the problem and indicate when customers can expect to hear from us next.

Explain what errors users can expect to see and how we believe the behavior of the service is impacted. That way, if any *new* problems arise customers will let us know instead of assuming it's part of an existing incident. If we have a workaround customers can use safely, let them know.

Our tone with customers should be **honest**, **courteous**, and **apologetic**.

Keep explanations high-level and stick to terms customers know. During the outage, the goal is to give customers enough information to demonstrate we know what’s going on -- or that we're working to find out. Customers can’t act on implementation details and this isn’t the time to explain the finer points of our systems. If we need to, we can follow up with more details in a customer-visible post-mortem or root cause analysis.

When we’re confident the outage is over and the problem will not recur, we should give customers the all-clear. The all-clear message lets customers know they can get back to work _and_ that they should contact us again if they see new or continuing problems.

If we _think_ the problem is solved but we’re still monitoring, say that instead.

# Example
**12:15pm**  
We’re investigating some problems with the service. We should have more information in the next 15 minutes. Sorry for the trouble.

**12:25pm**  
We’ve identified a configuration issue in the service. You may see 500 errors while updating stacks that include more than five resources. We’re working on a fix. Next update in 20 minutes.

**12:40pm**  
We’ve implemented a fix and we’re rolling it out now. We expect this to take about an hour. We’ll update here in 30 minutes. In the meantime, your stacks can be updated if you pass the --macguffin flag.

**1:10pm**  
The rollout of the fix is going well but taking a bit more time than we expected. Current estimate is it will be another 45 minutes. We’ll update here again in 30 minutes when we have a firmer timeline.

**1:30pm**  
We’re still monitoring the rollout. It should be done by 2:00pm. We’ll update here then.

**1:55pm**  
The fix has been rolled out everywhere. We believe things are back to normal but we’re doing some extra validation to make sure this addresses all variations of the problem. We’ll report back in 10 minutes with results.

**2:00pm**  
The fix is rolled out everywhere and we’ve verified it addresses the bug you were seeing. Your stacks are ready for use and no other action is required. We’re sorry again for the inconvenience. Please let us know here if you see anything else out of the ordinary.

# Other reading
[Pagerduty’s customer liason training](https://response.pagerduty.com/training/customer_liaison/)

Examples of good customer-visible post-mortems:
- [AWS S3, Feb 2017](https://aws.amazon.com/message/41926/)
- [Google Cloud, Apr 2016](https://status.cloud.google.com/incident/compute/16007)
- [Azure, Feb 2012](https://azure.microsoft.com/en-us/blog/summary-of-windows-azure-service-disruption-on-feb-29th-2012/)
