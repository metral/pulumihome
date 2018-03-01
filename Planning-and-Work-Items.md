We have some "ground rules" for planning and work item management.

### TL;DR: minimal label requirements

- **PRs that are a breaking change must have the `impact/breaking` label.** This always implies `impact/changelog`, so you don't have to use both labels.
- **PRs that have a user-facing change must have the `impact/changelog` label.** If a feature needs product documentation, it must  have this label. The PR should have a user-facing description of the change. This text will be copy-edited before adding to change log. So, don't worry about the right user-facing text, just make sure everything relevant is captured.
- **If an issue is closed without writing code**, use a `resolution` label.

### To also help keep issues orderly
- Use `help wanted` for issues that a community member could pick up. Ensure issue description is clear for someone who is new to the project
- An `area` label is suggested for all issues, but is not required
- A `kind` label is strongly suggested for all issues.

   | Label | Description |
   ---------|----------|
   | kind/bug | Product bug. If not user-facing, add the "kind/engineering" label | 
   | kind/design | Design notes, design idea |
   | kind/engineering | Work that is not user-facing. Makefiles, Travis, ops. Formerly known as `area/infra` |
   | kind/enhancement | User facing, by definition. Smaller than a feature, but an improvement of some sort |
   | kind/feature | User facing, by definition. Major new "thing" such as stack history, changes to promises model, etc |
   | kind/question | Used to label questions that customers ask | 

### Using issues

**All work must be tracked by work items, no matter how small.**  This helps with scheduling and release notes, which we are trying to semi-automate.

**All work items must have an assigned owner.**  At this stage, you should know who is the natural choice for any piece of work.  Assign it to them when creating the work item.  Assign it to them if you see one without an owner.  The assignment can change, of course, when scheduled, but the owner of the area is the de facto shepherd for the area and associated work.

**Any completed work item must be tagged in the current milestone.**  Sometimes we reach forward and pull in work items earlier than expected for various reasons.  Ideally, we would schedule that work item when realizing it will get done sooner.  In the worst case, however, the work item should be placed into the current milestone when closing it.  This ensures that we don’t miss something when generating our release notes.

**No person can be “overbooked” for a milestone.**  At the start of each sprint, count the number of business days (15 for a 3-week sprint), then add up your work items with their estimated days.  If that number exceeds the available time in the sprint, you are overbooked.  We must never overbook anybody.  We have been quite bad here and usually scramble to punt work items every sprint.  I’ve been letting it slide more than makes me comfortable -- and to be honest, GitHub doesn’t make this easy to manage! -- but for us to credibly know what it takes to ship our product with the required features and bug fixes, we will need to be disciplined here.  Thankfully, given the team’s background, you all know exactly what I mean.

**No work item can be <1d or >5 for purposes of planning.**  Anybody who has worked with Joe or Eric knows this rule.  On the average, very few work items take less than 1 day to finish, especially if the work is of high quality with good tests (which it must be), and especially given our multi-repo and service-oriented world.  And any work item that is more than 5 days screams one of two things: either “I don’t actually know what this work entails” or “multiple things got glommed into a single work item,” both of which can be remedied by splitting the work into multiple, smaller work items.

**Issues that need refinement can be labeled as such.** If you file an issue that is not ready to be implemented immediately, use the label `status/product-design` or `status/tech-design`. Once the design has been decided, change the label and add a comment to the issue to record the decision (or link to design notes).

## Full list of labels

| Label | Description |
---------|----------|
| area/cli | CLI |
| area/core | Engine |
| area/docs | API docs, user docs, etc |
| area/examples | Sample content |
| area/languages | Language providers | 
| area/providers | Resource providers | 
| area/service | Anything related to the Pulumi Service or managed stacks |
| area/tests | area/tests	Unit tests, integration tests, test automation, test breaks |
| area/tools     | Standalone tools that are **not** the CLI |
| customer/feedback | Feedback received from customers |
| customer/onboarding | Customer onboarding and customer success work |
| help wanted | Formerly named "status/job-jar". Issues that community can pick up. |
| impact/breaking | **Must be used on PRs that have are a breaking change**. Implies that it will be in the change log. |
| impact/changelog | **Must be used on PRs with user-facing impact and need to show up in the change log.** If a feature needs product documentation, it must also have this label. The first comment in the PR should have a user-facing description of the PR. This text will be copy-edited before adding to change log. |
| impact/performance	| Used on issues that track performance work |
| impact/reliability    | Lurking bugs. Might not be a bug now, but something error-prone for either the team or customers |
| impact/security | Used on issues and PRs that have a security impact |
| impact/usability | Usability of some aspect of the system |
| kind/bug | Product bug. If not user-facing, add the "kind/engineering" label | 
| kind/design | Design notes, design idea |
| kind/engineering | Work that is not user-facing. Makefiles, Travis, ops. Formerly known as `area/infra` |
| kind/enhancement | User facing, by definition. Smaller than a feature, but an improvement of some sort |
| kind/feature | User facing, by definition. Major new "thing" such as stack history, changes to promises model, etc |
| kind/question | Used to label customer questions | 
| priority/P0 | Used alongside release* labels. Indicates ship blocker for a release. |
| priority/P1 | Used alongside release* labels |
| priority/P2 | Used alongside release* labels |
| release/* | Used to track work for a particular release target, such as release/oss |
| resolution/duplicate | |
| resolution/norepro	 || 
| resolution/wontfix	| |
| status/product-design | Feature or area that needs to be designed from a product perspective|
| status/eng-design     | Feature or area that needs additional technical design. I.e., implementation is not obvious. | 


## Adding a new label

- Add the label to the list above
- Add the label to the `pulumi/pulumi` repo
- Check with Joe/Donna about label syncing process