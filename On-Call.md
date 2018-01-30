*Please revise with more details.*

# Schedule

Our on-call schedule is [here](https://docs.google.com/spreadsheets/d/1J-AWVK1F_VEvIq9K_5PJmHum69jb40DKkUMLeqdI7C8/).

# So, You're On Call?

If you're on the schedule, there are some things you'll need to know and do.

## Overview

At all times, there is a *primary* oncall person and a *secondary* oncall person to act as a backup.

Oncall is handed off on Tuesday mornings to coincide with our production releases. This week's secondary will be next week's primary.

If this is your first, please take 10 minutes to [read this](https://landing.google.com/sre/book/chapters/being-on-call.html).

Please plan in advance during sprint planning, as if you have on-call duty during your sprint, you will want to budget some time for performing "SRE duties" and proactive improvements to our tools and processes.

## Responsibilities

_These are still very much in flux._

### Primary

* Respond to pages and incidents. Triage issues, respond, and bring in the help you need.
* Shepherd this week's production release from staging
* Write post-mortems for outages that happen during your shift
* Triage notifications (`#ops-notifications` in Slack) once per business day

### Secondary

* Respond (as backup) to pages and incidents if oncall is unavailable
* Shepherd this week's staging release from testing

TODO: Define oncall SLO and escalation path

## PagerDuty

We use PagerDuty for escalations, and it keeps track of who will get notified. This is not automatically updated as people roll in and out of the rotation, so when you become primary on call, you *MUST* update this. To do so:

1. Visit [Pulumi's PagerDuty](https://pulumi.pagerduty.com/incidents), this will require that you log in.
2. In the right hand "Who is on call now?" box, click on the "Pulumi Oncall" link, to view information about the rotation.
3. There will be an "Edit Escalation Policy" box on the top right of the screen. If you do not see this, ask Joe to make you a "Manager".
4. Move yourself from secondary to primary and then add your backup as secondary on call.
5. Save the policy.

# Alerts Playbook

See https://github.com/pulumi/live-site for troubleshooting tips, etc.

# Inspiration

> "The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at and repair."
