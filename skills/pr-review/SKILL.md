---
name: pr-review
description: Independently review a contributor's pull request, verify behavior and decisions, and prepare actionable feedback and a human QA handoff.
---

# Review a contributor's pull request

Confirm the PR head and base, scope, design references, discussion and previous reviews. Inspect the complete diff, including new files and fixes. Verify any inferred issue association against the actual title and described behavior. Keep the contributor's worktree intact. If reproducing the change needs a build or fixtures, use an isolated checkout and an appropriate test environment.

## Reconstruct the decisions

Map each requirement to the implementation, including approved changes to the original plan. Flag undocumented choices with lasting consequences for users or maintainers. Check product terms, role permissions and interaction patterns against the target project's own guidance. When a decision's history is unavailable, ask for it or label it unknown.

## Test the case for merging

- Follow changed logic from a real entry point. Identify behavior removed along with the replaced code, and inspect old records or clients that still depend on the contract.
- Look for tests that reproduce the author's assumptions. Exercise a realistic counterexample and the error path most likely to affect users.
- Where the PR changes a screen, export or generated artifact, inspect the rendered result with representative fixtures. A compiled component alone cannot show whether someone can finish the task.
- Check deployed dependencies, operational steps, rollback implications and maintainability where the diff changes them. Support findings with a reachable trigger and concrete impact.
- For a re-review, identify the previous head, check each prior finding against the new revision, then inspect the new changes for regressions.

Return a merge recommendation and ranked findings with locations, consequences and proposed fixes. Give the human reviewer the few files or interactions most worth inspecting, the expected result, and any decision only they can make. Provide paste-ready review comments where useful. Post them only when asked.
