---
name: implement-phase
description: >-
  Run one phase of a Linear issue end to end without round trips: batch every
  open decision into a single question up front, then implement, verify,
  browser-test, self-review with /phase-check, fix, and commit. Use when
  starting a phase you have already planned. For auditing a spec before
  implementation use /spec-check, for a reviewer's pass use /code-review, for
  pre-merge use /cto-review.
argument-hint: "phase number or name, e.g. \"phase 2\" — plus the Linear issue if it isn't already in context"
effort: high
---

# Implement a phase

**Host tools.** Use the host’s `/code-review` for an independent diff review if available, or request an independent code review of the same commit range.

One phase, start to finish, in one turn.

The round trips this skill targets come from decisions that could have been batched up front and requests for permission already granted.

So: **ask everything once, then run.**

**Out of scope:** planning the phases (see below), auditing the issue before
implementation (`/spec-check`), a reviewer's pass (`/code-review`), merge
readiness (`/cto-review`).

### Plan mode first, or not

Plan mode and this skill answer different questions. Plan mode asks *what should
this phase build, and how*: an approach you can argue with before anything is
committed to. The decision gate in Step 1 asks *what is unspecified that I need
in order to execute an approach we already agree on*.

Enter plan mode first when you would want to argue with the approach: anything
design-heavy, anything spanning more than one screen or surface, anything where
a wrong structural call is expensive to unwind. Then hand the approved plan to
this skill.

Skip it when the approach is obvious and only details are open. A two-file
config change does not need a plan. It needs the decision gate, which on that
kind of work reliably catches more than a plan would, because it is aimed at the
repo's current state rather than at the shape of the solution.

Follow the target repository's instructions and permissions throughout. Identify operations it reserves for a human before starting. If a phase depends on one, flag that step and its owner.

---

## Step 0 — Load the phase

1. Read the Linear issue in full. It is the source of truth, and it may have
   been rewritten since the plan was made.
2. Identify this phase's boundary: what it includes, and what belongs to a
   later phase. Write both down.
3. Re-read the approved plan for this phase if one exists.
4. Confirm the branch and working tree state. If the phase begins fresh work,
   update from the project’s agreed base branch and start the work branch before editing.
5. Read enough surrounding code to know the patterns you will be matching.

State the boundary in two lines before going further. A phase that quietly
grows is the second most expensive failure here, after the round trip.

### Falsify the spec's factual claims before building on them

An issue describing a bug is making a claim about how the code behaves today.
Some of those claims are wrong, and the cheapest moment to find out is now.

For each one phrased as observed behaviour, spend up to ten minutes trying to
**reproduce it** — one browser interaction, one grep, one temporary probe. If it
does not reproduce, stop and say so; do not implement around it.

> Example: a test can force two resource IDs through the same component instance when the real router remounts it. Reproduce the alleged state leak by navigating between resources in the running app before changing the reset behavior.

This is not the decision gate below. That one asks what the issue leaves
*unspecified*; this asks whether what it *asserts* is true.

## Step 1 — The decision gate

**This is the point of the skill. Do it properly and the rest runs unattended.**

Before writing any code, find every decision this phase actually requires and
that the issue does not already settle. Look for:

- Copy that has to be written, not just placed.
- Behaviour on the paths the issue doesn't describe: empty, error, loading,
  offline, mid-flight, already-done, permission-denied.
- Anything a user sees that has no mock.
- Ordering or timing a user could notice.
- A dependency on state you would have to create, backfill, or fake.
- Anywhere the issue and the code disagree about what exists today.

Then put **all of them into one question batch**, using `AskUserQuestion` or the equivalent in your agent, with a recommended
option first. Batch even the ones you are fairly confident about: the marginal
cost of a fourth question in an existing batch is near zero, and the cost of
discovering it mid-implementation is a full round trip plus a context switch.

Ask about a scope change on its own, labelled **Scope change**. It needs a decision separate from the agreed work.

If you find nothing worth asking, say so in one line and continue. Do not
manufacture questions to fill the batch.

### The bar

**Stop and ask for:**

- Anything a user sees or reads that isn't specified: copy, labels, empty
  states, error text.
- Product behaviour the issue doesn't cover.
- A design decision with no mock, where more than one answer is defensible.
- Anything in the `AGENTS.md` hard-stop list.
- Work that genuinely widens the blast radius beyond the ticket's goal.
- The discovery that the spec is wrong, contradicts the code, or contradicts a
  decision made earlier in the thread. Say so immediately; don't implement
  around it.
- A real fork in engineering approach where the choice is hard to reverse
  later.

**Do not stop for:**

- Naming, file placement, or directory structure.
- A file the ticket failed to name that its correctness depends on. Edit it and
  say so in the report.
- Which existing pattern to follow. Follow the nearest one.
- Test structure, fixture choice, or how many tests.
- How to fix a review finding whose diagnosis you accept.
- Anything already decided in the issue, the plan, or earlier in this thread.
- Permission already granted to run checks, drive the browser or commit. Honor any approval the user has reserved.
- Reporting that you are about to start. Start.

Every call you make under "do not stop" goes in the final report, so it can be
vetoed cheaply. That is the trade: autonomy now, an audit at the end.

## Step 2 — Implement

Write the code. Match the surrounding conventions rather than importing your
own.

**Inner loop, after each meaningful edit batch.** Read the target project's development instructions, package or build scripts, and CI. Identify the correct working directories. Run its compiler or type checker and the tests related to the edited code. Save the full suite, uncached lint and build for the issue gate unless this repository requires them earlier.

**Run commands before documenting them.** Run the exact command from the documented working directory. If it is destructive or reserved for a human, identify the limit and leave the result unverified.

Check the environment flags, working directory and actual test selection. A mistyped selector can succeed while exercising nothing.

**The same rule applies to numbers.** Measure each timing, size and count you report. Wall time and summed parallel-worker time answer different questions. Count the artifacts the claim refers to.

Commit at meaningful checkpoints rather than only at the end. Each commit
message follows the repo's existing style and names the issue.

## Step 3 — The gate

Use two gates. Follow the target project's own policy if it calls for a stricter checkpoint.

### The phase gate — every phase

From the affected workspace, check formatting for files in scope, run the relevant compiler or type checker for typed code, and run the tests related to changed source. Identify the exact commands and any test environment configuration from the target repo's scripts and CI.

**Read the test count, not the exit code.** `vitest related` exits 0 when it
matches nothing, including for a mistyped path, so a silent green is the failure
mode this tier is most exposed to. A run with no file count is not a run.

Re-run only this tier after a `/phase-check` or `/code-review` fix, and only the
part the fix can reach.

### The issue gate — once, when the last phase is done

Before the PR and `/cto-review`, run the project's full formatting check, uncached lint, compilation or type checks, and full test suite. Run the production build for routing, configuration or runtime-boundary changes, and whenever CI requires it. If the change touches shared packages, run the checks for those packages and their consumers from the correct workspace. An app-only check can miss package tests.

This is where a cross-file breakage surfaces if `related` missed it. Accepting
that it surfaces here rather than mid-issue is the trade: it is still before
review and before CI, and it costs one run instead of one per phase.

Use the same uncached lint and pass/fail thresholds as CI. If an apparently unrelated warning or type error appears, compare with the base revision before attributing it. Report pre-existing failures and keep fixes within the issue's scope.

## Step 4 — Browser QA

Verify in the real app, against this phase's acceptance criteria, one criterion
at a time. Take the dev server as given if it's up; start it if it isn't.

For visual behavior, take a screenshot. Inspect ancestor visibility as well as the element you changed.
- **Read the screen as prose**, top to bottom, in the order a user reads it.
  Contradictions live between strings, not inside them.
- Check every viewport the phase claims to support, not just the one you had
  open.
- Batch independent browser checks when the available browser tool supports it.

If a criterion cannot be verified locally, say which one and why. Do not
silently substitute a weaker check.

## Step 5 — Self-review

Run `/phase-check` over this phase's diff, including any fix commits.

Fix what it finds. A finding is yours to fix, not to report and leave. If a
finding is genuinely a judgment call, it goes to the user in the report rather
than being decided quietly.

Re-run the affected part of the phase gate after fixing. Include every fix commit in the range handed to independent code review.

## Step 6 — Report

**Never report the state of anything you have not just observed.** A background
agent, a running command, a job you launched: call `ListAgents` (or check the
task output) before saying it is running, finished, or found nothing. Skills that
load instructions into the turn are the specific trap — loading one is not
running it, and an interruption between the two is easy to miss from the inside.

A claim about an agent or job needs a current status observation. Distinguish loading a skill's instructions from actually executing its checks.

Keep it short and factual. Six parts:

1. **What shipped.** The phase's scope, and confirmation the boundary held.
2. **Evidence.** What you verified in the browser, and how. Screenshots for
   anything visual.
3. **Gate results.** Report compiler or type errors, lint warnings, test-file and test counts, and the pass/fail result against the project's CI thresholds.
4. **What `/phase-check` found**, and how each item was resolved.
5. **Judgment calls I made without asking.** The list, each in one line. This
   is the part the user is actually reading; make it complete rather than
   flattering. Anything you decided under the "do not stop" bar belongs here.
6. **What I did not verify**, and the commit range for `/code-review`.

Then stop. Do not start the next phase, open a PR, move the Linear issue, or
merge unless asked. Ending the turn is the point at which the user gets to
steer, and they have earned a real decision rather than a fait accompli.
