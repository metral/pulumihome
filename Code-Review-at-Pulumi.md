## The goals

Code review spreads knowledge, improves quality, and avoids silos. We're all in this together.

## The policy
**All code at Pulumi must be reviewed and approved by another engineer before it's checked in.** That includes infrastructure code and configuration (e.g. a Dockerfile or .travis.yml).

There is one exception: comment-only changes of no more than a few lines may be submitted without review.

In an emergency -- for example, responding to a production outage -- you can use your judgment and submit code to be reviewed later. That said, don't underestimate the value of a second perspective, especially in a crisis. Also, don't forget the "reviewed later" part.

### Guidelines for authors
- Write your code with review in mind: make small changes and write commit messages that explain your motivation and intent.
- Include tests in the same pull request as the code that needs them.
- Choose a reviewer whose time will be well-spent on the review. That can mean a lot of things:
  - Pick a reviewer who knows the code well and can do a thorough review quickly.
  - Pick a reviewer who's ramping up on the code, so they can ask questions and learn things they'll need to know.
  - Pick a reviewer who caught the last bug you almost shipped, since they're that much more likely to catch the next one.
  - Pick someone new because you're doing something weird and need a different perspective.
- Address all feedback: make changes or push back.
- If there is a time constraint, let your reviewer know.

### Guidelines for reviewers
- Treat code reviews as high priority. If you don't think you'll be able to turn a code review around in 24-48 hours, let the author know early.
- Review for design, implementation, and style, in that order.
- If you have trouble following a change, say something! Authors are responsible for making their code understandable. If you think something is obscure or just a bit too clever, there's a good chance that future readers will agree with you.
- Approve a change only if you would be comfortable maintaining it.
  - Would you be okay swapping the "Author" and "Reviewer"?
  - Would you feel confident being on-call for a service running this code?
  - Are there enough tests to convince you the code works? Will those tests catch the most likely regressions?
- For larger discussions, always consider talking in person, but make sure your new understanding makes it back into the written code review discussion.
- If a disagreement boils down purely to personal preference, tie goes to the author.
