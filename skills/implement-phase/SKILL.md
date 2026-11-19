---
name: implement-phase
description: Implement one agreed phase of a feature, including targeted checks, real-flow verification, self-review, and handoff. Use after the phase and approach are defined.
---

# Implement one phase

Read the current issue or brief, its amendments, the approved plan, and the repository instructions. Identify the phase boundary and its observable acceptance criteria. Inspect branch and working state so existing work is preserved. Find the applicable test, build and lint commands from this checkout's configuration and CI.

## Resolve decisions before editing

For a reported bug, try the smallest reproduction at the entry point the user actually uses. If the premise fails, investigate the discrepancy before building a fix. Gather the remaining decisions that affect product behavior or are expensive to reverse into one concise request with recommended defaults. Use established project conventions for reversible details. Treat extra deliverables as scope changes.

## Build and verify

Work in cohesive increments, following nearby patterns. Exercise the smallest relevant checks after each batch and inspect which tests actually ran. Run the repository's required full checks at its prescribed checkpoint. If a failure appears unrelated, compare against the same baseline before attributing it.

For each acceptance criterion, exercise the normal interface with suitable test data. For a UI, follow the real navigation and inspect the rendered screen at affected sizes. For an API, CLI or library, use its public entry point and inspect the resulting state or output. Include the failure or retry path that motivated the change. Record any environment limitation against the specific criterion it prevents you from verifying.

## Review and hand off

Review the complete diff, including fixes made during review. Look for displaced behavior, untested callers and mocks that hide the original defect. Recheck the affected behavior after fixing it. Follow the user's commit instructions and project conventions, including any reserved approval.

Report delivered behavior, verification actually performed, findings fixed or still open, judgment calls and remaining limits. Identify the diff or commits the reviewer should inspect. A completed phase is the handoff point for the next decision.
