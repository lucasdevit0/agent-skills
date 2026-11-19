---
name: phase-check
description: >-
  Author-side pre-flight on a diff you just wrote, checking for
  displaced behavior, adversarially re-read evidence, grep-shaped audits,
  tests that inherit the author's blind spot, copy read out of sequence,
  labels that name a destination they never reach, and unreviewed fix commits.
  Use after finishing a phase or commit, before /code-review. Runs on YOUR OWN
  diff from the author's seat — for a reviewer's pass use /code-review, for a
  spec use /spec-check, for pre-merge use /cto-review.
argument-hint: "commit range, or omit to use uncommitted changes plus this phase's commits"
effort: high
---

# Phase check

**Host tools.** Use the host’s `/code-review` for an independent diff review if available, or request an independent code review of the same commit range.

The gates are green and you are about to hand this to review. This is the pass
that asks what the gates structurally cannot.

Run these checks even when the compiler, linter, tests and browser smoke checks pass. Each examines behavior those checks can miss.

The distinguishing property: each is a failure of *what you verified*, not of
*whether you verified*. That is why more care doesn't fix them and a checklist
does.

**Out of scope:** reviewing the diff from a reviewer's seat (`/code-review` —
run it after this), spec readiness (`/spec-check`), merge readiness
(`/cto-review`).

**No subagents.** This is a first-person pass over your own work. Delegating it
reproduces the blind spot, because the agent inherits your framing.

## Step 0 — Gather

1. The diff: the argument's range, else `git diff main...HEAD` plus
   `git diff HEAD` for uncommitted work. Also list the commits in range —
   several checks are per-commit, not per-diff.
2. The claims you made. Scan this conversation for what you asserted about this
   work: "verified", "confirmed", "no consumers", "nothing else uses". Class O
   re-reads these.
3. If a plan drove this work, re-read it. Class S has a habit of originating
   there.

## Step 1 — Scope the classes

Not all apply. Classify, state what you're skipping and why in one clause each.
Skipping is fine; skipping silently is not.

- **Deletes or replaces anything** — a component, a wrapper, a prop, a branch,
  a whole file? → **N**
- **Made any factual claim about the codebase, or ran any verification?** → **O**
- **Found or fixed instances via a search?** → **P**
- **Changes behavior?** → **Q**
- **Changes anything a user reads?** → **R**
- **Touches a control, link, or button?** → **S**
- **Any commit in range fixes review findings or a bug?** → **G**

## Step 2 — The checks

### N — The displacement ledger
For every deletion or replacement, write down what the removed thing was
*providing* — not what it was called. Then confirm each item is either replaced
or deliberately dropped, out loud.

The trap is that you verify the removal succeeded. Nobody verifies what left
with it.

**Write the ledger as a lifecycle, not a snapshot.** Two questions per removed
mechanism, because the second is the one that gets skipped:

1. What did it *render or expose* while it was active?
2. What did it do **on state change** — clear, reset, revalidate, unsubscribe,
   invalidate, tear down?

Anything that participated in a framework's own cycle was getting behaviour for
free. Replacing it with state you own means you now own the teardown too, and
nothing will remind you: the happy path looks identical and only the *second*
interaction is wrong.

> Examples: hiding a desktop panel can remove the only heading on mobile. Dropping a shared form wrapper can remove its cancel link. Replacing a form library's error state with a local flag can reproduce the red border and disabled submit but lose automatic clearing on revalidation. Test the next interaction as well as the first render.

### O — Your own evidence, re-read adversarially
For each verification you ran, ask: **what would this output look like if the
thing were broken?** If the answer resembles what you actually got, you have
not verified it.

A probe scoped to one element may miss an ancestor's visibility or a border that fails to render. Screenshot visible changes.

> Example: a count of zero visible headings passes a “no duplicates” check. It fails the actual requirement of exactly one visible heading.

**Every guard test ships with a control that fails.** Not "when it feels
load-bearing" — always, for any test asserting that something *doesn't* happen,
or that a guard holds, or that a mutation would be caught. Write the paired case
that must produce the opposite result, and watch it produce it.

A passing test proves the assertion held. It does not prove the harness could
have observed the alternative, and those are different claims. The failure mode
is silent: green all the way to review, on a test that was never able to fail.

> Examples: a test rerenders an existing instance on a parameter change while the router mounts a new instance. A negative async assertion passes before the effect runs. A global dialog stub drops the close callback, so every test passes when asserting Escape is ignored during a request. Drive Escape while idle too: if the dialog cannot close in that control, the harness cannot demonstrate the guard.

### P — Grep-shaped audits miss the other spelling
If you found the instances by searching, name the other ways the same thing can
be written, and search for those too. State the spellings you covered.

> Example: a search for toast notifications finds raw error strings in banners but misses equivalent strings passed to a form's inline error handler. Trace the underlying error to every display path.

### Q — Tests that inherit the author's blind spot
A test written from the same mental model as the code confirms intent instead of
probing it. For each behavioral change, name the input where your implementation
and the spec could **disagree**, and assert on that one.

Three smells: every test uses the same fixture value; the new control/branch has
no test at all while its sibling does; and a **test double shaped to the calls
you happen to make**. A stub satisfying the one method under test makes every
other access silently wrong, and it is the code you did not write that the real
object has to support.

> Examples: a validation fixture uses a character accepted by both conflicting rules. A recovery form tests the retry button but skips its neighbor. A query-parameter mock implements reads and writes but omits iteration, hiding data loss when code copies the parameters. Use representative inputs and a real instance where the contract matters.

**Then re-read every fixture you wrote and ask: would this still pass if my rule
were wrong in the obvious direction?**

A rule that decides between two outcomes has two ways to be wrong, and a fixture
written from the rule only ever exercises the half you were thinking about. Your
tests prove the branch *fires*; nothing yet proves it stays quiet on the inputs
next to it. Name the nearest input that should take the other branch, and assert
that it does.

Mutation testing does not save you here: mutating the rule breaks the tests you
already wrote, which are all on the same side, and it goes green.

> Example: a parser picks a category whenever it finds a keyword anywhere in a label. Bare keywords exercise only positive matches. Try an input where the same word occurs in a note about a different item. A useful negative fixture survives the same code path and must retain its original category.

### R — Read the screen as prose
Read every string the user sees, top to bottom, in one pass, in the order they
appear. DOM probes assert strings individually; contradictions only exist
*between* them.

Applies doubly to copy you wrote late, and to text shared across screens: read
it in situ on **every** consumer, not just the one you had open.

> Examples: a success message promises immediate access, followed by instructions to sign in manually. A shared subtitle asks for an email address on a later screen that has no email field. Read both strings in context.

### S — A label that names a destination must reach it
For each control, read its label as a promise and check the handler keeps it.
Flag any two controls on one screen wired to the same handler: one of them is
probably mislabelled, or one is redundant.

Check the plan too. This class originates in specs more often than in code, and
a wrong plan converts to a wrong implementation with every gate green.

> Example: a navigation link invokes the same state reset as a nearby retry control. A prototype sharing one handler between both buttons does not specify that they have the same destination.

### G — Fixes are the least-reviewed code
*(Inherited from `/spec-check`, which reserves this class and cannot check it.)*

Commits that fix review findings, or fix a bug, are the ones nobody re-reviews —
the review already happened. For each such commit in range, confirm it is inside
the range you are about to send to `/code-review`.

> Example: a reviewer inspects the original range while the fixes for that review sit in a newer commit. Verify the head of the next review includes both.

## Step 3 — Report

Per class: **clean**, **fixed** (say what), or **skipped** (say why). Then:

1. **What I did not verify.** The honest list. A gate being green is not
   verification of anything but the gate.
2. **The range for `/code-review`** — including any fix commits per G.

Findings here are yours to fix before review, not to report and leave. If a
finding is genuinely a judgment call, say so and put it to the user rather than
deciding quietly.

One rule for the report: do not describe a check as clean because it passed the
form you happened to run. State what the check *was*, then its result. Class O
exists because that distinction was collapsed once already.
