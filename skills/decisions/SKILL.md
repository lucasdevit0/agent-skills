---
name: decisions
description: >-
  Identifies open product, technical, and UX decisions needed for the current
  project or scope of work, then provides recommendations for the user's review.
  Use when the user asks for /decisions, decision review, open questions,
  decisions to make, or wants the agent to surface decisions in advance so an
  AI agent can work independently — including scope creep risks, hardening vs
  deleting choices, UX patterns that conflict with the app or with best
  practices, and technical smells or gotchas.
disable-model-invocation: true
---

# Decisions

## When this applies

The user wants `/decisions` or an equivalent: surface every meaningful decision blocking confident, autonomous AI work on the current scope, then recommend a default for each.

## Instructions

Are there any product, technical, or UX decisions that need to be made for this project or scope of work? Please identify the questions and then provide your recommendations for my review.

Pay careful attention to the types of decisions that should be made in advance so that an AI agent can work successfully independently.

Watch out for scope creep, decisions about hardening vs. deleting, UX patterns that are in contrast to standard best practices (or other UX patterns that we have in the app already), as well as technical smells or gotchas that might come back to get us later.

## Discovery process

1. Establish the scope. Use the latest user prompt, any linked Linear issue, recent diffs, and open files. If scope is ambiguous, restate what you believe it covers and ask the user to confirm before continuing.
2. Read the relevant code, shared schemas and enums, handlers, migrations and access policies. Trace behavior across the boundary the decision affects.
3. Compare against existing patterns in the app — UX components, form patterns, route handlers, naming conventions, error/empty states — so you can flag anything that diverges.
4. Note unresolved TODOs, feature-flagged behavior, half-built features, and dead/legacy code in the touched area. These are common sources of "harden vs delete" decisions.

## What counts as a decision worth surfacing

- **Product / scope** — what is and isn't included; MVP vs full version; target user/role; success criteria; rollout (flagged, gradual, full).
- **UX** — primary flows, empty states, error states, loading states, permissions/role differences, mobile behavior, accessibility, anything diverging from existing app patterns.
- **Technical** — data model and schema choices, RLS posture, API contract shape, sync vs async / background jobs, idempotency, caching, error handling, observability, retries.
- **Hardening vs deletion** — when an existing feature, path, or table is touched, decide whether to invest in fixing it or remove it. Flag dead or legacy code rather than silently leaving it.
- **AI-agent readiness** — anything that would otherwise force the agent to guess: vague acceptance criteria, undefined enums, missing fixtures, naming choices, or interdependent choices that should be locked first.
- **Smells / gotchas** — N+1 patterns, missing indexes, `any` cascades, env-var assumptions, race conditions, untested error paths, migration ordering risks, breaking changes to public types.

## Output format

Group findings by category in this order: **Product / scope**, **UX**, **Technical**, **Hardening vs deletion**, **AI-agent readiness**, **Smells & gotchas**. Skip any category that genuinely has nothing — do not pad.

Number the decisions continuously across all categories, starting at 1 and never resetting per category, so every decision has a stable reference number (e.g. "decision 3"). Put the number in the decision heading.

For each decision use this template:

```
### N. [Short title for the decision]
- Question: What needs to be decided, in one sentence.
- Why it matters: Concrete impact (blast radius, who's affected, what breaks or gets locked in if guessed wrong).
- Options: 2–4 realistic choices with the key tradeoff for each.
- Recommendation: The default you'd ship and why.
- Confidence: High / Medium / Low — and what would raise it.
- References: File paths with line ranges, migrations, or doc/Linear links.
```

End with a short **Open questions for you** checklist — just the questions, as a numbered list reusing each decision's number, in priority order — so the user can answer in one pass by number.

## Safety rules

- Keep permission, database and implementation changes out of this review. Flag the choice and recommend a default.
- Do not start implementing — this skill is decision-surfacing, not execution.
- If you are not confident a decision actually matters for this scope, leave it out.
- Prefer fewer, sharper decisions over a long undifferentiated list.

## Anti-patterns

- Listing decisions the user clearly already made earlier in the conversation.
- Recommending "it depends" with no default.
- Restating the diff instead of identifying what is still ambiguous.
- Ignoring existing app patterns and recommending a new one without naming the conflict.
- Quietly expanding scope inside a recommendation instead of calling it out as a scope-creep decision.
