# Schedule

The weekly livesite summary document is [here](https://docs.google.com/document/d/12-0rsxf3qMdUvRvHSSHg5SvGv_dKILprPWnc5HLCAeU/edit#)

Our on-call schedule is [here](https://pulumi.pagerduty.com/schedules).

We have [[instructions on how to add oncall to your calendar automatically|Adding your oncall schedule to Google Calendar]].

You can add these WebCal feeds to Google Calendar to see who's oncall right now:

| Rotation | WebCal feed |
| --- | --- |
| Primary | webcal://pulumi.pagerduty.com/private/1ac6154c1561df2337fde2dec1f8660d396ba4dca9690606573ee2ad91661761/feed/PDW8FBN |
| Secondary | webcal://pulumi.pagerduty.com/private/1ac6154c1561df2337fde2dec1f8660d396ba4dca9690606573ee2ad91661761/feed/PKO4HT1 |

# Overview

We run a service that customers use and rely on.  When the service isn't working right or customers can't use it, we want to know, we want to fix it, and we want to keep that particular problem from happening again.

Pulumi's oncall rotation makes sure there is a clear path of responsibility to respond to issues when they arise.  Beyond the calendar, the oncall rotation is also about making sure the people oncall have the resources, authority, and support they need to respond with confidence.

Our thinking about oncall is heavily influenced by [this chapter of Google's SRE book](https://landing.google.com/sre/book/chapters/being-on-call.html). Take a chance to read it before your first shift -- or your fiftieth.

# Mechanics

At all times, there is a *primary* oncall person and a *secondary* oncall person to act as a backup.

Oncall is handed off on Tuesday mornings to coincide with our production releases.  One week's secondary will be the next week's primary, so that they start their primary shift with context on the state of the service.

New engineers will *shadow* oncall for a week before entering the rotation as primary or secondary. 

Please plan in advance during sprint planning, as if you have on-call duty during your sprint, you will want to budget some time for performing "SRE duties" and proactive improvements to our tools and processes.

# Responsibilities

_These are still very much in flux and incomplete. Please take the time to review and edit during your own shift._

## Primary

* Respond to pages and incidents. Triage issues, respond, and bring in the help you need.
* Handle or delegate [customer communication](https://github.com/pulumi/home/wiki/Customer-communications-during-an-incident) of incidents or planned downtime.
* Shepherd [releases](https://github.com/pulumi/home/wiki/Weekly-release-process) from master to staging and staging to production
* Write post-mortems for outages that happen during your shift
* Triage notifications (`#ops-notifications` in Slack) once per business day

## Secondary

* Respond (as backup) to pages and incidents if oncall is unavailable

## Everyone else

* Help oncall when they ask
* Let oncall know about potentially disruptive changes or testing

TODO: Define oncall SLO and escalation path

# PagerDuty

We use PagerDuty for escalations, and it keeps track of who will get notified. This is not automatically updated as people roll in and out of the rotation, so when you become primary on call, you *MUST* update this. To do so:

1. Visit [Pulumi's PagerDuty](https://pulumi.pagerduty.com/incidents), this will require that you log in.
2. In the right hand "Who is on call now?" box, click on the "Pulumi Oncall" link, to view information about the rotation.
3. There will be an "Edit Escalation Policy" box on the top right of the screen. If you do not see this, ask Joe to make you a "Manager".
4. Move yourself from secondary to primary and then add your backup as secondary on call. **Make sure "escalate after" is set to 5 minutes.**
5. Save the policy.

# Response playbook

See https://github.com/pulumi/live-site for troubleshooting tips, etc.

When responding to incidents, we need to keep customers in the loop. See [guidelines and examples here](https://github.com/pulumi/home/wiki/Customer-communications-during-an-incident).

# Inspiration

> "The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at and repair."
