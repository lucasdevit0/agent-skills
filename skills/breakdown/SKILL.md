---
name: breakdown
description: Turn agreed product requirements into ordered implementation phases, acceptance criteria, and a builder handoff. Use when a feature needs a technical plan or issue breakdown.
---

# Break requirements into shippable phases

Read the agreed requirements, designs and any related work in progress. If the outcome is still unsettled, identify the specific question that prevents a plan. Proceed on independent requirements when possible.

## Trace the work

For each requirement, find the actual entry points, owners of state, shared contracts, consumers and tests. Verify that existing patterns you intend to reuse handle the required behavior. Check adjacent work for duplicate effort or sequencing conflicts. Mark decisions the builder can safely own and surface product choices that require the owner's input.

## Order the phases

Aim for one independently reviewable pull request per phase. Walk through the system after each merge, including a rollback while old and new versions coexist. If a phase exposes a control before its dependencies work, change the order or describe the rollout mechanism. A prerequisite can ship internally before the feature becomes visible.

For each phase give a title, deliverable, affected surfaces, dependencies, observable acceptance criteria, risk and any human-operated steps. Estimate relative size from the shape of the change if useful. Keep the cost of a mistake separate from implementation effort.

Map each requirement to a phase. Inspect any unmapped requirement and any phase with no requirement. Read the resulting plan as someone new to the work. Could they find the first entry point, obtain test fixtures, exercise an error path and tell whether they are done?

## Hand off

Name the first phase, setup and access needed, available test data, verification steps, and owners of release actions. Write issue drafts so each stands alone. Spell out decisions and reasons rather than referring to a conversation.

When tracker updates are requested, use the project's existing tracker conventions. Check for existing phase issues before creating new ones, record real dependencies, link them from the plan, and read back the result. A partial write can be resumed from the issues already created.
