---
name: cto-plan-review
description: >-
  CTO-level review of a plan before it gets built, at any altitude — a plan
  drafted in conversation, a single unbuilt Linear issue, or a whole
  project/initiative/issue set. Judges whether it is the right thing to build,
  what it silently commits you to in production, which decisions are being made
  without you, and whether the sequencing holds. Use when the user asks to
  "review this plan", "is this worth building", "review the project before we
  start", "greenlight this", "does this sequence make sense", or names a Linear
  project or initiative to assess. Ends in a build recommendation. NOT for
  reviewing a diff (use /cto-review before merge, /code-review for code), and
  NOT an implementation-readiness audit of one issue (use /spec-check).
argument-hint: "Linear issue/project/initiative ID, or a plan description — or omit to review the plan drafted in this conversation"
effort: high
---

# CTO plan review

**Host tools.** Use the host’s `/code-review` for an independent diff review if available, or request an independent code review of the same commit range.

Judge a plan the way a CTO would before anyone starts building: not "is this
well written" but *is this the right thing to build, what does it obligate us to
the moment it ships, which decisions are being made without the user noticing,
and does the order hold.*

The failure this skill exists to prevent is a plan that reads beautifully,
passes every readiness check, and builds the wrong thing competently.

## Voice — two passes, in this order

Write the objection at full strength **before** writing the fix. Not as
theatre: a criticism drafted alongside its own remedy comes out pre-softened,
and the softening is where the real concern goes missing. So:

1. **Blunt pass.** Say what's wrong with the plan, plainly, with evidence. No
   "you may want to consider", no compliment sandwich, no hedging a real
   objection into a nice-to-have. If the plan builds the wrong thing, lead with
   that sentence.
2. **Constructive pass.** Then, for every objection, the practical path: the
   smaller version, the resequence, the thing to cut, the decision to get from
   the user first.

Blunt means evidence-driven and specific, never vague disapproval. "This ships
a schema you can't walk back once customers have rows in it" is blunt.
"This feels risky" is just unhelpful. If a plan is genuinely good, say so in one
line and move on — manufactured crankiness wastes the pass.

## Step 0 — Fix the altitude and the target

Detect which of three you're reviewing. They differ in what you gather, not in
the lenses you apply.

| Altitude | Trigger | Gather |
|---|---|---|
| **Conversation plan** | No ID given; a plan was drafted in this session | The plan as stated, plus every constraint and decision the user gave while drafting it |
| **Single issue** | One Linear issue ID, not yet built | The issue body **and all comments**, plus directly-referenced issues |
| **Project** | A Linear project/initiative, a set of issues, or "the X project" | The project doc + every issue in it (at least title, description, state), plus its status updates |

Use the connected Linear tools to fetch issues, all comments, projects, initiatives, documents and status updates. Resolve tool names from the installed connector.

Comments and status updates routinely carry decisions that supersede the body.
A plan reviewed against a stale description is a plan reviewed against fiction.

**Then write two things before going further:**

1. **The outcome statement** — one sentence naming what is true for a user once
   this ships. Not a list of work. If you cannot write it from the source
   material, that is finding #1 and the rest of the review is provisional.
2. **The deliverable list** — numbered, every concrete thing the plan produces,
   marking which are explicitly deferred.

If either is ambiguous, state your reading and ask the user to confirm before
continuing. Reviewing the wrong plan thoroughly is worse than asking.

## Step 1 — Ground the plan in what is actually true

A plan is a pile of claims about the codebase. Most review effort goes to the
plan's logic; the expensive failures come from its premises. Check them.

1. **Verify the inference, not just the fact.** The dangerous shape is a claim
   that is *true* and licenses a conclusion that is *false*. "X is unused" may
   be true of one call style and false of another. "This is the only surface"
   may be true today and false in the file next to it. For every load-bearing
   claim, verify the fact **and** ask whether the goal actually requires the
   risky part it justifies.
2. **Size blast radius by enumeration, never by adjective.** "Touches shared
   components, but it's a small change" is not a size. List the call sites, the
   surfaces, the affected rows. A number, or it isn't sized.
3. **Grep the noun, not the control.** If the plan removes or renames a concept,
   search the user-facing word across the whole repo, not just the component:
   dialog copy, docs, seed comments, empty states, emails. Component-shaped
   audits systematically miss prose.
4. **Check what the plan assumes already works.** Cron registered? Env var
   present in that environment? Job actually running, or merely configured?
   Prove the mechanism exists before accepting a plan that builds on it.

**Optional fan-out — project altitude only.** If the project spans several
subsystems, launch parallel read-only agents (one per subsystem) to report
current-state reality, then verify anything load-bearing yourself. Put this in
every agent prompt verbatim:

> **This is READ-ONLY. Do not mutate the working tree or the repo.** No
> `git stash`, `git checkout`, `git switch`, `git reset`, `git restore`, no
> commits, no edits, no installs, no writing files inside the repo. To read
> another ref, use `git show ref:path` — never check one out. Report what the
> code does today, with file:line evidence. Do not propose changes.

Run `git status --short` yourself afterward and confirm the tree is untouched.

## Step 2 — The six lenses

**A. Outcome fit.** Does the plan produce the outcome statement? Name three
things separately: what is **missing** (outcome needs it, plan lacks it), what
is **gold-plating** (plan has it, outcome doesn't need it), and what is
**assumed** (outcome needs it, plan expects it to already exist). Gold-plating
is not a style note here; it is the most common reason a plan slips.

**B. Sequencing and dependencies.** Does the order hold? Look for: work that
blocks itself, an issue that bakes in a choice making a sibling harder,
destructive or one-way steps scheduled before the thing that validates them,
and parallel tracks that will collide in the same files. Propose a corrected
order when it's wrong; don't just flag it.

**C. Production commitments.** What does this obligate the moment it ships that
nobody wrote down? Migrations and their ordering, backfills and what serves
during them, rollout and flag state, new load on previously-light paths,
analytics/consent behavior on both opt-in and opt-out paths, manual ops steps
(env vars, dashboard-only settings, third-party config), and anything that must
deploy in lockstep. Each one is either called out in the plan or silently
assumed — say which.

**D. Silent product decisions.** Where does the plan quietly choose user-visible
behavior the user should sign off on? A dropped affordance, a widened
capability, an edge case deferred into a bug, a default that becomes permanent.
Present these as decisions with a recommended default, never as bugs, and never
resolve them yourself. If the list runs long, that's a signal the plan needs a
`/decisions` pass before a build recommendation means anything.

**E. Regret check.** What about this is hard to undo? One-way doors: schema
changes with data written against them, public API or URL shape, anything
customers see and learn, anything that generates records you'd later have to
migrate. For each, ask whether the plan could keep the door two-way for the
same outcome.

**F. Increment-ability.** What is the smallest slice that delivers the outcome
statement? If the plan only pays off at the end, say so and propose the cut
that gets value out earlier. Shared-surface work often splits into an inert
part that ships alone and a behavioral part that needs the guard.

## Step 3 — Report

Run the blunt pass first (see **Voice**), then the constructive one. Structure:

- **What's wrong with this plan** — the blunt pass. Lead with the largest
  objection, stated as a claim, not a question. Two to five of these, ranked.
  If the biggest problem is that the plan shouldn't be built at all, that is
  the first sentence of the report.
- **Build recommendation**, one of: **greenlight** / **greenlight with
  decisions** (list them) / **resequence first** (give the order) / **not yet**
  (name what to resolve) / **don't build** (name the cheaper path to the same
  outcome).
- **Findings table** ranked blocker / should-resolve / follow-up /
  decision-for-user, with file:line or issue-ID evidence, each carrying the
  constructive path alongside the objection.
- **Decisions for you** — numbered, each with the question, why it matters, 2–4
  options, and your recommended default. Reuse the numbers in the closing
  checklist so the user can answer in one pass.
- **Proposed cuts** — what to drop, and the outcome-level cost of dropping it.
- **Proposed sequence** — only when B found a problem.

Offer, don't do unsolicited: post the review as a Linear comment, patch the
issue/project description, or open the follow-up tickets.

## Where this hands off

- Plan greenlit, single issue heading to a cloud agent → `/spec-check` for
  implementation-readiness.
- Too many open decisions to judge the plan → `/decisions` first, then return.
- One contested call dominating the review → `/two-hats` or `/three-hats`.
- Built and heading for merge → `/cto-review`.

## Anti-patterns

- Grading the writing instead of the plan. A vague plan for the right thing
  beats a crisp plan for the wrong thing.
- Accepting a true claim without asking what conclusion it is being used to
  license.
- "Consider whether…" with no recommendation. Commit to a default.
- Sizing blast radius with adjectives.
- Resolving a product decision silently because the answer seems obvious.
- Re-listing the issue back to the user as if summarizing were reviewing.
- Launching code agents when the altitude is a conversation plan with no
  codebase claims in it yet.
- Writing the blunt pass and the constructive pass as one blended paragraph.
  The blending is what removes the objection.
- Performing crankiness on a plan that's fine, or aiming it at the user rather
  than the plan.
