---
name: three-hats
description: >-
  Argue one contested decision from three independent perspectives: a staff
  engineer, a head of product, and a senior product designer. Then commit to a
  single recommendation. Use when the technically-correct, product-correct, and
  design-correct answers might differ: scope cuts, "is this worth the blast
  radius", defer vs do now, fix-properly vs ship-narrow, taking debt vs taking
  risk, shipping a screen in a half-designed state. Runs on ONE decision, not a
  list. For surfacing many open decisions at once use /decisions, for auditing a
  spec use /spec-check, for pre-merge use /cto-review. For a faster two-voice
  pass with no design lens use /two-hats.
argument-hint: "the decision, e.g. \"should the checklist collapse after the last step or stay visible?\" — or omit to use the one under discussion"
effort: high
---

# Three hats

One decision. Three genuinely independent arguments. One recommendation at the end.

The value is *not* a balanced survey. It's that the three perspectives care about
different things, so between them they surface a consideration a single pass
misses. Most often that's a **fourth option** none of "yes", "no", or "later"
contained.

All three hats carry equal weight. None of them has a veto, including the
designer. Any of them can lose the argument.

## Phase 0 — Frame the decision

1. State the decision in one sentence, as a fork. If you can't reduce it to a
   fork, it isn't ready for this skill: say so and ask what's actually being
   chosen.
2. If the user's framing rests on a premise that's wrong or misremembered,
   **correct it before arguing**. A decision argued from a false premise wastes
   the whole exercise. This is common when a decision was partly settled earlier
   and the recollection has drifted.
3. If there are several decisions in play, pick the one that most constrains the
   others and say why you picked it. Offer to run the rest separately.

## Phase 1 — Get the facts first

No hat gets to speculate about things that can be checked. Before arguing,
verify the load-bearing claims: what else uses this, what breaks if it's wrong,
what test would catch it, what the current behavior actually is, what the screen
renders today.

Print the evidence. A single grep can collapse a long debate: an argument about
"we'd be touching invite" is over the moment you find that invite imports the
helper rather than defining it.

The designer's facts are checkable too, and usually are not checked. Before
Phase 4, know:

- Which existing primitive or pattern already covers this. Inspect the target project's component library and design system before recommending a bespoke component.
- What the non-happy states currently render: empty, loading, error, partial,
  and the non-admin / no-permission view. Read them, don't assume they exist.
- What the surrounding steps in the flow do, so continuity claims cite real
  screens.

Do not proceed to Phase 2 on assumptions. If something can't be verified, mark
it explicitly as unverified and let all three hats account for the uncertainty.

## Phase 2 — Staff engineer

Argue from technical judgment and consequences, not elegance. Work through:

- **Blast radius.** What changes that nobody asked to change? Name the specific
  surfaces, from the codebase, not from intuition.
- **Verification story.** How would we know this broke? Is there a test, or is it
  "someone eyeballs it"? Absence of coverage is an argument in itself.
- **Reversibility.** If this is wrong, what does undoing it cost: a revert, a
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
- **What the user sees.** Concretely. If the answer is "nothing", say so: that
  reframes the whole decision.
- **Core vs periphery.** Is this risking the thing customers pay for in order to
  improve the wrapper around it? That trade is almost always bad.
- **Cost of deferring.** Often zero. Check rather than assume.
- **Opportunity cost.** What doesn't get built, and does that matter more?
- **Who's affected.** All users, new users, one segment, internal only?
- **Coherence of shipping alone.** If this lands by itself, does the user see a
  change that makes sense, or an unexplained one?

## Phase 4 — Senior product designer

Argue from craft, comprehension, and continuity. Not from taste.

Identify the people using this flow from the project's product context. Check their task, familiarity, frequency of use and time pressure. For an infrequent user facing a deadline, judge whether they could complete the task without exploring the interface or reading recovery instructions.

Work through:

- **System coherence.** Does this match what the app already does, or does it
  introduce a second way of doing the same thing? Name the existing pattern it
  should reuse, by file. A one-off that duplicates an existing primitive is a
  real cost even when it looks fine in isolation.
- **The states nobody designed.** Empty, loading, error, partial, stale,
  zero-permission, and the very-long-content case. Which of these does this
  change create, and which one currently renders as nothing or as a raw error?
  This is the single most common thing the other two hats miss.
- **Comprehension.** Read the actual labels and copy out loud, in sequence, as
  the user would meet them. Does the label name what happens, or what the code
  does? Would a first-time user know what to do next without being told?
- **Recoverability.** If they misunderstand this and click it, what does it cost
  them, and can they get back? Weight destructive and irreversible actions much
  more heavily than anything cosmetic.
- **Accessibility as correctness, not polish.** Keyboard path, focus handling
  after the thing appears or disappears, contrast, hit targets, whether state is
  conveyed by colour alone, what a screen reader announces. Treat a failure here
  as a defect, not a nice-to-have.
- **Whole-flow continuity.** Judge this in the journey it lives in, not as an
  isolated screen. Follow the actual preceding step, the changed interaction, and the next action or output. Look for repetition, contradiction and references to things that have moved or disappeared.
- **Motion and transition.** If something appears, changes, or completes, does
  the change read as one continuous thing or as a flash? Only argue this where
  the transition carries meaning; don't spend the hat's credibility on easing
  curves.

Guard against these, or the hat is worthless:

- Preference stated as user harm. "I'd have done it differently" is not an
  argument. Tie every claim to comprehension, recovery, coherence, or continuity.
- Redesigning instead of deciding. The job is to argue the fork, not to propose a
  new screen. If the design really is the wrong shape, say that in one sentence
  as its own finding and keep arguing the fork.
- Gold-plating a decision that has no UI surface. Say "no user-visible surface,
  nothing for this hat" in one line and move on. Padding it out invents tension
  the recommendation then has to unwind.
- Title inflation. Senior product designer, not design director. The role is a
  lens on what this specific change does to a specific person using it, not
  design strategy or brand.

## Phase 5 — Find the fourth option

Before recommending, spend real effort here. The fork as stated is often false.
Ask:

- Can it be scoped narrower and still deliver the user-visible part?
- Can it split into an **inert** part and a **risky** part, shipping the inert
  part now?
- Can the risky part be made trivially reversible, so the decision stops being
  expensive?
- Is there a version that takes the debt instead of the risk, or vice versa?
- Is there a version that keeps the design coherent by **removing** an
  affordance rather than adding an explanation? Cutting the confusing control is
  frequently better than captioning it.
- Does the fork disappear if one unhandled state is designed? Sometimes "ship vs
  don't ship" was really "the error state doesn't exist yet".

If a fourth option beats all three branches, that's the recommendation, and say
plainly that the original fork was the wrong question.

## Phase 6 — Recommend

Commit. One option, stated as a decision, not a menu.

Say which hat you're overruling and why. With three voices the temptation to
average is stronger; averaging produces something none of them argued for.

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
## Senior product designer
## Where they diverge      ← name the real split; or "all three land in the same place", and why
## Recommendation
What it costs · Which hat I'm overruling · What would change my mind
```

Keep each hat to a few tight paragraphs. Length is not rigor. Three hats is a
wider lens, not a longer document.

## Anti-patterns

- **The same opinion in three voices.** If the sections make the same argument
  with different vocabulary, you haven't run three passes. Go back and find what
  each role would genuinely weigh differently. With three hats this failure is
  more likely, not less, and the designer section is usually the one that
  collapses into the product section. Design coherence and user outcome are not
  the same claim.
- **Manufactured disagreement.** If all three land in the same place, say so
  immediately and move to the recommendation. Convergence is a strong signal;
  don't dilute it by inventing tension.
- **Splitting the difference.** A compromise nobody would have chosen on purpose
  is worse than any single branch. Three-way averaging is the worst version of
  this.
- **The designer as tiebreaker.** The third hat exists to add a lens, not to
  break 1-1 ties. If the engineer and the product lead split, decide it on the
  merits, not on which one the designer stood next to.
- **"It depends."** If it genuinely depends, say what it depends on and pick the
  branch you'd take absent new information.
- **Deferring to the user mid-argument.** They asked for a recommendation. Give
  one, then let them overrule it.
- **Arguing from unchecked facts.** Phase 1 exists because this is the failure
  that wastes the most time. It applies to the design claims too: read the
  component, don't imagine it.
- **Title inflation.** Staff engineer, not principal or distinguished. The role
  is a lens for concrete technical consequence, not organizational strategy;
  inflating it produces posturing instead of reasoning.
