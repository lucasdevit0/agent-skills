---
name: spec-check
description: >-
  Pre-implementation audit of a Linear issue for surface parity misses, unsized blast
  radius on shared code, unverified or true-but-misleading claims, unspecified
  async/navigation behavior, destructive-side-effect ordering, and drift-prone
  text. Use when the user asks to /spec-check, harden a spec, check an issue
  before implementation, or asks whether a ticket is ready to hand to a cloud
  agent. Audits whether a spec is READY TO BUILD, not whether it's the right
  thing to build — for that judgment use /cto-plan-review. Runs against a SPEC,
  not a diff — for a diff use /code-review, for pre-merge use /cto-review.
argument-hint: "Linear issue ID or the spec drafted in this conversation"
effort: high
---

# Spec check

**Host tools.** Use the host’s `/code-review` for an independent diff review if available, or request an independent code review of the same commit range.

Audit the spec against the following failure classes before anyone writes code.

A spec that passes is one a cloud agent can implement unsupervised.

**Out of scope:** code correctness (that's `/code-review`), merge readiness
(that's `/cto-review`), and defect class G — "fixes are the least-reviewed
code" — which is an implementation-time rule and cannot be checked here.
`/phase-check` owns G, and the rest of the implementation-time classes (N, O,
P, Q, R, S) that only exist once there is a diff.

## Step 0 — Gather

1. Resolve the issue from the argument, the branch name, or the conversation.
   Fetch it **and all its comments** via the connected Linear tools. Fetch related and blocking issues the description references.
2. Collect decisions made in **this conversation** — they are spec content that
   hasn't landed in the description yet.
3. Restate the spec as a numbered deliverable list. Mark explicit deferrals.
   If you cannot produce this list, the spec is too vague to check: say so and
   stop.

## Step 1 — Scope the risk classes

Not every check applies. Classify the change first:

- **Touches a surface that renders an entity appearing in more than one place?** → F
- **Changes something shared** — a design token, the global stylesheet, a base
  layout, a common component, a shared helper, an env/config value? → M
- **Adds or changes an async user action?** → A, C, D
- **Adds UI state to a page reused across entities** (dialog, draft text, in-flight flag)? → B
- **Has two or more side effects, one irreversible** (delete + abort upload, DB write + S3 delete, email)? → E
- Always run: H, L, I, J, K.

State which classes you're running and which you skipped, with one clause of
reasoning each. Skipping is fine; skipping silently is not.

## Step 2 — The checks

Each check is a **question the spec must already answer.** If it doesn't, that's
a gap. Do not guess the answer — surface it as a decision.

### F — Surface parity
Grep for every surface that renders the same entity or offers the same action.
Do not reason from the spec's own file list; search the codebase. The spec must
name each surface as **in scope** or **explicitly out**, with a reason.
> Example: deleting an item from a collection view works, but the global search view still offers a different delete action for the same record.

### M — Shared-surface blast radius
`F` asks which *screens* render the entity. `M` asks what else consumes the
*code* being changed — a token, a base stylesheet rule, a common component, a
shared helper. Four questions, in this order:

1. **Is it sized, or just named?** "Touches body text app-wide but it's one
   coherent change" is an adjective, not a size. The spec must **enumerate** —
   name the surfaces, count the call sites. Search for consumers; don't take the
   spec's word for the list.
2. **Does the stated goal require it?** The highest-leverage question here,
   because it fires before any code exists. If the goal is served by scoping the
   change to the surface that actually needs it, that is usually the better
   spec. Ask it plainly and recommend an answer.
3. **What's the guard, and is it mechanical?** Prefer a check that produces a
   binary answer — diff the compiled stylesheet, diff the generated types, diff
   the built bundle — over "a visual pass shows no broken layout," which is
   unbounded and unfalsifiable. If a mechanical guard exists, it belongs in the
   acceptance criteria *instead of* the eyeball, not alongside it.
4. **Can the inert part ship separately?** Shared-surface work often splits into
   an additive part with provably zero effect and a retargeting part with all
   the risk. If a subset compiles to byte-identical output, say so — it can land
   immediately and shrink what needs real review.

> Example: a plan changes a root font token to restyle one form. Inspect everything that inherits the token, including headings with an explicit override. Consider scoping the font to the form and reviewing the global migration separately.

If the spec is phased, each phase should make plain **what the reviewer is
actually judging** and which phases carry the risk. Naming phases by
architectural role is fine and often clearer — the point is that a reviewer
opening phase 1 knows whether they're checking a token addition or an app-wide
retype, not that the title encodes it.

### E — Destructive-before-outcome
For each irreversible effect: does the spec state that it runs **only after**
the primary outcome is known? What happens to it if the primary fails?
> Example: canceling an attachment upload before confirming deletion loses the in-flight file if deleting the record fails.

### B — Per-navigation reset
For each new piece of UI state: does the spec say it resets when the user
navigates to another entity on the same reused page?
> Example: a dialog opened for record A stays open after navigation to B. Check dialog state, errors and pending requests separately.

### A — Identity source
For each mutating action: does the spec name the authoritative source of "which
entity" — the URL param, not a fetched object that lags behind it?

### C — Guard scope
If the spec requires double-submit protection, does it say the guard is **per
entity**, not per hook or per page?
> Example: a stale request releases the guard for a newer operation.

### D — Failure reporting vs navigation
If the spec specifies a failure toast on an async action: does it say what the
user sees when the action fails **after** they navigated away?
> Example: a navigation check returns before the error handler, leaving the user unaware that deletion failed.

### H — Unverified claims
Flag every factual assertion about the codebase: "no callers", "dead code",
"unused endpoint", "the migration is pending", any count. **Verify each one
now** and print the evidence, or mark it UNVERIFIED in the spec.
> Example: a sibling ticket calls a migration pending. Verify the current state using the project's permitted evidence before treating it as a blocker. A search hit or commented-out line cannot establish the live value.

### L — The inference behind the fact
`H` asks whether each claim is **true**. `L` asks what the spec **concluded**
from it, and whether that follows. This is the more expensive failure, because a
true premise passes every review that reads the spec rather than the code.

For each load-bearing fact, write the inference as a separate sentence and check
it independently. Two patterns worth naming:

- **Declared ≠ active.** A font, CSS variable, feature flag, provider, config
  key or middleware can be registered and still affect nothing. Verify the
  *effect* — the computed style, the resolved value, the actual request — not
  the declaration.
- **Symmetric brokenness reads as correctness.** Two things can agree because
  both are broken the same way. Repairing one splits the pair. Before fixing one
  member of a matched set, ask what else is matched to it *by accident*.

> Example: two layouts register a font variable without applying its family. Both look the same because the fallback is active. Compare computed styles and compiled output before changing only one layout.

Reviews inherit premises. When a fact is doing real work in the argument, the
cheap move is to run something — compile it, print the computed value, hit the
endpoint — rather than read harder.

### I — Drift-prone text
Flag and propose removal: line numbers, issue IDs destined for code
comments, and hard counts (error totals, row counts, file counts). All three go
stale between spec and merge.

### J — Decision completeness
Every open question has an answer in the description or an explicit deferral.
Answers buried in comments count as gaps: the description is the source of
truth, and comments are not read cold after a weekend.

Includes control behavior: where the spec names a button or link, it must say
where that control **goes**, not only what it is labelled. A label is not a
destination, and an implementer will wire whatever the spec implies.
> Example: the spec gives a link a destination label but assigns it the same in-place reset as its neighbor. State the destination of each control. See `/phase-check` class S.

### K — Handoff readiness *(only if this may go to a cloud agent)*
- Single well-defined outcome + acceptance criteria
- Named files or modules, no line numbers
- A **manual-QA checklist** — the human steps the agent cannot perform
- Assign operations the target project reserves for humans to the responsible person. Keep those operations out of unattended implementation.

## Step 3 — Report

```
READY / READY WITH GAPS / NOT READY
```

Then, in this order:

1. **Gaps needing a decision** — numbered `D1…Dn`, each with: the class, the
   question in one line, and **your staff-engineer recommendation**. Written so
   they can be answered inline, in one pass, without opening the codebase.
2. **Verified claims** — what you checked and what it actually said.
3. **Proposed description patch** — the exact text to add or cut. Do not write
   it to Linear yet.

### Update the Spec status label

After reporting the verdict, set the issue's **Spec status** label to match — it's the at-a-glance signal of whether the issue is safe to hand off:

| Verdict | Label |
| -- | -- |
| READY | `Ready` |
| READY WITH GAPS | `Ready with gaps` |
| NOT READY | `Seed` |

`Spec status` is an exclusive group. Resolve its labels by ID in the issue's team, read all current labels, replace only the member of this group, and send the full set when `save_issue` replaces labels. Preserve all other groups, including estimates. If the team uses equivalent names, match by meaning. If it has no equivalent, report the verdict and ask how to map it before changing the workspace taxonomy. Update the label again if the description patch resolves the gaps. Skip the label write for a conversation-only draft.

Then stop and ask. After the answers come back, patch the **description**, not
the comments, and confirm what changed.

Scope note: when a gap is small and lives in code already being touched, prefer
folding it into this ticket over filing a follow-up. Deferred work on a solo
team does not get done. Widen scope only when it genuinely widens risk, and say
which it is.

The mirror of that: when a ticket contains a **risky item the stated goal
doesn't require**, splitting it out is not deferral-by-another-name — it's
letting the rest ship on a guard that actually proves something. Recommend the
split when the risky part can be scoped to the surface that needs it, and when
what remains is provably inert. Say plainly what the split costs; there is
usually an interim inconsistency, and it's usually worth it.
