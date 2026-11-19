---
name: two-hats
description: >-
  Argue one contested decision from two independent perspectives — a staff
  engineer and a head of product — then commit to a single recommendation.
  Use when the technically-correct answer and the product-correct answer might
  differ: scope cuts, "is this worth the blast radius", defer vs do now,
  fix-properly vs ship-narrow, taking debt vs taking risk. Runs on ONE decision,
  not a list — for surfacing many open decisions at once use /decisions, for
  auditing a spec use /spec-check, for pre-merge use /cto-review.
argument-hint: "the decision, e.g. \"should we load Inter app-wide or scope it to auth?\" — or omit to use the one under discussion"
effort: high
---

# Two hats

One decision. Two genuinely independent arguments. One recommendation at the end.

The value is *not* a balanced survey — it's that the two perspectives care about
different things, so between them they surface a consideration a single pass
misses. Most often that's a **third option** neither "yes" nor "no" contained.

## Phase 0 — Frame the decision

1. State the decision in one sentence, as a fork. If you can't reduce it to a
   fork, it isn't ready for this skill — say so and ask what's actually being
   chosen.
2. If the user's framing rests on a premise that's wrong or misremembered,
   **correct it before arguing**. A decision argued from a false premise wastes
   the whole exercise. This is common when a decision was partly settled earlier
   and the recollection has drifted.
3. If there are several decisions in play, pick the one that most constrains the
   others and say why you picked it. Offer to run the rest separately.

## Phase 1 — Get the facts first

Neither hat gets to speculate about things that can be checked. Before arguing,
verify the load-bearing claims: what else uses this, what breaks if it's wrong,
what test would catch it, what the current behavior actually is.

Print the evidence. A single grep can collapse a long debate — an argument about
"we'd be touching invite" is over the moment you find that invite imports the
helper rather than defining it.

Do not proceed to Phase 2 on assumptions. If something can't be verified, mark
it explicitly as unverified and let both hats account for the uncertainty.

## Phase 2 — Staff engineer

Argue from technical judgment and consequences, not elegance. Work through:

- **Blast radius.** What changes that nobody asked to change? Name the specific
  surfaces, from the codebase, not from intuition.
- **Verification story.** How would we know this broke? Is there a test, or is it
  "someone eyeballs it"? Absence of coverage is an argument in itself.
- **Reversibility.** If this is wrong, what does undoing it cost — a revert, a
  migration, a customer conversation?
- **Debt vs risk.** Is this taking permanent risk to avoid temporary debt, or
  temporary debt to avoid permanent risk? Name which.
- **Review surface.** Who reviews this, and will they understand what they're
  approving? A correct change buried inside an unrelated PR is reviewed by nobody.
- **Failure mode.** What happens on the unhappy path, not the happy one.
- **Correctness pull.** Watch for conflating "the right thing" with "the right
  thing *now*". Both can be true; they're different claims.

## Phase 3 — Head of product

Argue from user outcome and opportunity cost. Work through:

- **The stated goal.** Name it explicitly, in the user's own words if they gave
  them. Then answer plainly whether this serves it. "No" is a valid and useful
  answer.
- **What the user sees.** Concretely. If the answer is "nothing", say so — that
  reframes the whole decision.
- **Core vs periphery.** Is this risking the thing customers pay for in order to
  improve the wrapper around it? That trade is almost always bad.
- **Cost of deferring.** Often zero. Check rather than assume.
- **Opportunity cost.** What doesn't get built, and does that matter more?
- **Who's affected.** All users, new users, one segment, internal only?
- **Coherence of shipping alone.** If this lands by itself, does the user see a
  change that makes sense, or an unexplained one?

## Phase 4 — Find the third option

Before recommending, spend real effort here. The fork as stated is often false.
Ask:

- Can it be scoped narrower and still deliver the user-visible part?
- Can it split into an **inert** part and a **risky** part, shipping the inert
  part now?
- Can the risky part be made trivially reversible, so the decision stops being
  expensive?
- Is there a version that takes the debt instead of the risk, or vice versa?

If a third option beats both branches, that's the recommendation, and say plainly
that the original fork was the wrong question.

## Phase 5 — Recommend

Commit. One option, stated as a decision, not a menu.

Then state honestly:
- **What it costs.** Every recommendation gives something up. Name it.
- **What would change your mind.** A concrete fact or condition, not a hedge.

## Output

```
## The decision
One sentence. Plus a premise correction if one was needed.

## Facts checked
What you verified and what it actually said. Evidence, not assertion.

## Staff engineer
## Head of product
## Where they diverge      ← or "Both land in the same place", and why
## Recommendation
What it costs · What would change my mind
```

Keep each hat to a few tight paragraphs. Length is not rigor.

## Anti-patterns

- **The same opinion in two voices.** If both sections make the same argument
  with different vocabulary, you haven't run two passes. Go back and find what
  each role would genuinely weigh differently.
- **Manufactured disagreement.** If both hats land in the same place, say so
  immediately and move to the recommendation. Convergence is a strong signal —
  don't dilute it by inventing tension.
- **Splitting the difference.** A compromise nobody would have chosen on purpose
  is worse than either branch.
- **"It depends."** If it genuinely depends, say what it depends on and pick the
  branch you'd take absent new information.
- **Deferring to the user mid-argument.** They asked for a recommendation. Give
  one, then let them overrule it.
- **Arguing from unchecked facts.** Phase 1 exists because this is the failure
  that wastes the most time.
- **Title inflation.** Staff engineer, not principal or distinguished. The role
  is a lens for concrete technical consequence, not organizational strategy;
  inflating it produces posturing instead of reasoning.
