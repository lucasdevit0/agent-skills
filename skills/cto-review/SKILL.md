---
name: cto-review
description: Review built work against the agreed scope and production consequences. Use before merge when an owner needs a release recommendation.
---

# Make a pre-merge recommendation

Establish the proposed merge range, target branch, issue or brief, amendments and explicit decisions from the owner. Confirm which revisions you inspected. If there is no built change, identify the missing diff and direct the conversation to a plan review.

## Review from three angles

1. **Delivery.** Map each promised outcome to implementation and evidence. Mark complete, partial, missing or deliberately deferred. Identify changes with no corresponding requirement.
2. **Product and operations.** Walk through the first deployment, intermediate data states and a rollback. Which users see different behavior, which workers or services run the new code, and which manual steps must happen in order? Inspect build and release configuration before claiming that merge triggers a deployment.
3. **Fresh code read.** Trace the highest-impact change from its real caller through persistence or output. Check permissions, failure recovery, compatibility and tests where relevant. Verify the reach of shared code with actual consumers. If independent reviewers are available and appropriate, give them the same scope and revisions and verify their findings against the code.

Keep the review read-only. Compare other revisions without disturbing active work. Use an isolated checkout for checks that write files. Distinguish a plausible risk from an observed failure and size the impact with the evidence available.

Lead with merge, merge after specific conditions, or hold. Give each blocker a triggering case, effect, code location and resolution. Surface decisions the owner needs to accept and deployment actions someone needs to own. Record the checks actually run and any unverified release assumptions.
