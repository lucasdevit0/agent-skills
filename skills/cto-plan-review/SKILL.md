---
name: cto-plan-review
description: Review a proposed feature, issue, or initiative before implementation. Use to decide whether to build it, reduce scope, resolve product choices, or change the sequence.
---

# Review a plan before building

Write the promised user outcome in one sentence. Identify the plan, constraints, prior decisions and evidence the outcome matters. Review at the right level. An early idea may need a product argument, while a migration plan needs a reading of existing callers and release order.

## Examine the proposal

- **Outcome.** Which requirements are missing, which depend on something assumed to exist, and which can be cut with little effect? Try to disprove the main assumption. A technically accurate statement can still support a poor conclusion.
- **Order.** Walk the intermediate states between phases. Find work that blocks itself, competing changes to the same contract, and points where users can enter a half-built flow.
- **Operations.** Look for unowned rollout, backfill, configuration, load or support work. Identify who experiences the consequence and when. Quantify reach where evidence allows.
- **Decisions.** Identify defaults and behavior changes the plan silently chooses for users. Give the owner a recommended option with its cost.
- **Reversibility.** Trace what rollback would leave behind, especially data, public contracts and user expectations.
- **Smallest release.** Describe the smallest coherent slice that delivers the stated outcome, or a cheaper way to test the assumption first.

Lead with the strongest concrete objection, if one survives verification. Give one recommendation: build, build after named decisions, change the sequence, wait for specific evidence, or pass. Pair each material objection with a feasible change and its cost. Keep estimates and unverified risks labeled. End with the decisions the owner can answer in one pass.
