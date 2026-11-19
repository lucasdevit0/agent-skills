---
name: pr-review
description: >-
  Review someone else's PR (a contractor, a teammate, or an unattended agent)
  before merge. Like /cto-review, but read-only and written for the
  reviewer: it traces the decisions the author made to what they will do to
  customers later, checks every product and UX call against the project's product
  docs, runs the /phase-check failure classes from the reviewer's seat,
  weighs long-term maintainability, and
  says exactly where a human should put eyes — which code to read and which
  screens to QA. Ends in a merge recommendation, decisions for the maintainer, and PR
  comments ready to paste. Use when the user asks to "review this PR", "review
  a teammate’s PR", "review a contractor’s PR", or passes a PR number that isn’t
  their own work. NOT for your own diff (use /phase-check, then /cto-review),
  NOT for a plan (use /cto-plan-review), and NOT for assessing the author as a
  hire.
argument-hint: "<PR number> [--since <sha>]"
effort: high
---

# PR review

Someone else wrote this PR. The maintainer has to decide whether it merges and what to say
to the author. Your job is to make that decision cheap: what was built, what it
commits us to, what conflicts with what we already know, and the short list of
places a human has to look for themselves.

Three differences from `/cto-review`, and they are the point of this skill:

1. **Read-only.** You never fix anything. Findings become PR comments or
   questions for the maintainer. The author fixes their own PR.
2. **Decisions over defects.** The author made calls the spec didn't make. Find
   each one and trace it forward. The bugs that matter most here are the
   decisions that look fine today and come back later, especially those that
   change what a customer sees or does.
3. **Human attention is the output.** End with where the maintainer's eyes should go,
   not just with what you found.

## Step 0 — Gather

1. **The PR.** Run `gh pr view <n> --json title,author,body,baseRefName,headRefName,headRefOid,commits,files,comments,reviews,statusCheckRollup`,
   `gh pr checks <n>`, and fetch all pages of inline comments from the base repository's pull-request comments endpoint (`repos/{owner}/{repo}/pulls/<n>/comments`)
   for inline review threads (`comments` only returns the conversation). Read
   the body, the commit messages and every comment: authors put their questions
   and caveats there. Below, `<base>` is the PR's `baseRefName`, usually `main`.
2. **A worktree at the PR head**, so agents can run the gates:
   ```
   git fetch origin <base> +pull/<n>/head:refs/remotes/pr/<n>
   git worktree add --detach <review-directory> refs/remotes/pr/<n>
   ```
   Run from the main checkout. Choose a separate review directory and a named ref so force-pushes and rebases can be followed. For an existing dedicated review worktree, check it is clean before advancing it to the fetched head. Install locked dependencies with the project's package manager so resolution uses this revision. For Agent E, configure the documented test environment, including any paid or external services needed by the cases. Keep production secrets out of the review worktree.
3. **The Linear issues.** Resolve them from the PR title, body and commits. Check the issue title and project as a branch can contain a project number resembling an unrelated issue ID. For each issue, fetch the
   description, **all comments**, and its blocking relations. Fetch the project
   description too: technical plans live there. Fetch any linked docs, such as
   an onboarding doc with process rules.
4. **Blockers.** For every "blocked by" or "must merge before" relation, check
   whether that work is in this PR or already on `origin/<base>` (after the
   fetch above). A missing blocker is a
   sequencing finding even when the author listed it in an unchecked box.
5. **The context set.** Find the target project's agent instructions, product overview, domain guidance, permissions model, design conventions, customer personas and relevant technical guides, including any migration, analytics or data interpretation notes.
6. **The scope list.** Number every acceptance criterion, every decision in
   comments, and every process rule from the onboarding doc. Mark which
   decisions the spec made and which it left to the author ("builder's call").

### Unrecorded decisions: ask while the agents run

Collect every place the PR claims a decision that isn't written down. Typical
signs include a claim about an unrecorded clarification, a behavior that contradicts an acceptance criterion, or a process change such as one PR for all phases. Launch the reviewers, then ask the maintainer about these items while they run. Forward confirmed answers to active reviewers or recheck the affected items yourself. Leave drift provisional until the maintainer answers.

## Step 1 — Parallel review agents

Before launching, save `git status --short` for the review worktree and for
the checkout you started from, so you can tell afterwards whether anything
changed.

Launch A, B, C and D, plus E when it applies (see below), in ONE message
(subagent_type: general-purpose). For a small PR (roughly under 100 lines of
non-test change, and no product context, background pipeline or database schema changes), do Agent C's check yourself instead, and say
so under Findings. Give each agent the worktree path, the diff command
(`git diff origin/<base>...HEAD` in the worktree; local `main` is often stale),
the scope list, and the decisions the maintainer has confirmed. Paste the context inline;
don't make agents re-fetch Linear.

> ### Put this in every agent prompt, verbatim
>
> **The review is READ-ONLY. Do not mutate the working tree or the repo.** No
> `git stash`, `git checkout`, `git switch`, `git reset`, `git restore`, no
> commits, no edits, no `npm install`, no writing files inside the repo.
>
> To compare against another ref, read from it directly with `git show <base-ref>:<path>`
> — or build into a temp directory outside the repo. Never check out a ref to
> look at it.
>
> Running the project's own read-only checks (`tsc --noEmit`, `eslint`,
> `prettier --check`, `vitest run`) is fine, and so is build output in
> gitignored paths (`.next/`, `*.tsbuildinfo`, tool caches). Anything else that
> writes inside the repo is not.
>
> Do not query production from a review agent. If sizing needs production data, return the proposed read-only aggregate query and the relevant data interpretation assumptions. The lead reviewer and maintainer apply the target project's production access rules.
>
> Never trust the PR description, commit messages or code comments: verify every
> claim against the code. Size every finding (who hits it, how often, what
> breaks); an unsized finding is a hypothesis, so label it that way.

**Agent A — Completeness.** For every scope item: DONE (file:line), PARTIAL,
MISSING, or DEFERRED-OK. Then re-verify **each factual claim in the PR
description**, such as "N tests pass", "matches the baseline" or "checked in a
browser", and say which ones held. List every change outside the scope list
(drive-bys, new emissions, seed changes, `.gitignore`, dependencies) and whether
it's justified.

**Agent B — Decisions and their consequences.** List every decision the author
made that the spec didn't, plus every spec decision whose implementation has
consequences nobody named. For each, trace it forward:
- **Customers.** What does a customer see, export, receive or lose because of
  it? Check every output path, not only the one the PR changed: editor, PDF,
  Word, email, approval, public pages.
- **Existing customers on merge.** Enumerate every entry point that reaches the
  new behavior, and say whether any existing customer's output changes without
  someone acting.
- **Later.** What does this commit us to, such as a stored format, a default, a
  term, an event or a permission? What gets hard to undo once customers have
  data in it? What breaks when the next obvious feature lands?
- **Ops.** Inspect how the application, workers and other consumers deploy. Which revision runs on merge, what happens between releases, which environment variables or migrations are needed, and which analytics funnels could shift?
- **Risks accepted earlier.** Search migrations, code comments and runbooks
  near what the diff touches for accepted risks with a revisit condition ("accepted
  because…", "if a later phase…", "revisit when…", "safe while nothing reads
  this"). Report any condition this PR meets. A column that was harmless while
  nothing read it stops being harmless in the PR that starts reading it.
Tag each one: author's call, spec's call, or forced by the code.

**Agent C — Context alignment.** Check product, UX, copy and domain decisions against the target project's documented sources of truth. Quote both sides of a conflict.
- Treat edits to authoritative product context as their own findings. Quote before and after, and name the person or role that owns sign-off.
- Distinguish domain principles from product requirements. Check a rewritten rule against its primary domain source. Implementation alone cannot establish the rule.
- Find product terms, plan names and roles that conflict with the canonical vocabulary or duplicate definitions maintained elsewhere.
- Compare behavior with the documented permission model, design conventions and personas. Include any segment with stricter requirements.

**Agent D — Fresh-eyes code, through the /phase-check classes.** Read
[the phase-check skill](../phase-check/SKILL.md) and apply its classes from the reviewer's
seat to the hand-written logic (skip fixtures and generated files). Each finding
needs file, line, a concrete failing input, and CONFIRMED (reproduced, for
example with a script in `/tmp` or a test run) or PLAUSIBLE:
- **N, displacement:** what did each removed or replaced thing provide,
  including teardown on state change, and was it kept?
- **O, evidence:** would the author's tests and baselines fail if the thing were
  broken? Check that "unchanged" fixtures were generated from the base
  branch's code, by rendering its version from `git show origin/<base>:` into
  `/tmp`.
- **Q, tests sharing the author's blind spot:** name the nearest input that
  should take the other branch, and check that a test asserts it.
- **R, copy as prose:** read every user-facing string in order, on every screen
  that shows it.
- **S, labels and destinations:** does each link or button go where its label
  says?
- **P, grep-shaped audits:** where the PR claims it fixed or updated every
  instance of something, name the other spellings and search for those too.
- **G, fix commits:** list commits that respond to review or fix bugs, and
  review each one as new code against everything, not only the finding it
  answers.
- **Permissions at every layer.** Check whether a lower layer permits what an upper layer denies. Inspect effective database grants and policies, direct data API access, privileged service identities and client-only checks. Verify anonymous access to new tables against the target project's actual configuration.
- **Sweep.** When you find a defect, search for the same pattern elsewhere in the
  diff and its callers, and report every site.
- **Long-term maintainability.** Read the diff as the next person to change it
  will: someone who wasn't in this conversation, months from now. Flag what
  will make that change slow or risky:
  - a rule now written in more than one place that has to change together, and
    whether the codebase already writes it inline elsewhere (matching an
    existing idiom isn't a finding);
  - code that diverges from how the surrounding code does the same thing
    (naming, file placement, error handling, data access);
  - new abstractions, options or flags with one caller, and dead or redundant
    branches;
  - tests that will be expensive to keep: large frozen fixtures, hand-built
    copies of real schemas or data shapes that can drift, and assertions on
    implementation details instead of output;
  - comments that restate the code, or make claims about other files that
     aren't true, and docs that now describe
    behavior incorrectly;
  - coupling that widens blast radius, such as pipeline-imported shared code
    gaining new dependencies.
  Size each one by what the next change costs, not by taste. Most land as
  follow-up or optional; promote one to "before merge" only when it is cheap
  now and expensive once customers depend on it.
Run the target project's verification commands in the review worktree. Include the root or package-level type checks used by CI, full lint, formatting, and tests related to changed paths. Check the selected test-file and test counts. Compare the actual results with the PR checks and note what was particularly well done for the maintainer's review.

**Agent E — Real-output QA** (launch with the others when the PR changes
anything a user sees, downloads or receives, such as a screen, an export or an
email preview; skip it otherwise and say so). It runs the PR's code on its own dev server and
produces the real artifact for each case: an exported file, a screenshot, a
rendered page. Give it the cases, taken from the acceptance criteria: one per
branch of the change, plus the unchanged case. Tell it:
- Start the PR's dev server on a distinct port. Override the app origin and any callback or renderer URL so exports and requests reach this worktree. Check that email and other external effects are captured locally.
- Use an isolated browser session or host and a test data store. Check cookie, storage and database isolation independently.
- Never reset or reseed a shared database for review. Prefer reads. For unavoidable writes to existing authorized test rows, capture the old state first, restore it afterwards, and report affected rows and recovery steps.
- If the data a case needs doesn't exist locally, skip the case and say so.
  Don't change fixture state to create it.
- Save each artifact to the scratchpad. Stop the dev server when done.

Then **open the artifacts yourself** (for example, unzip a .docx and read
`word/document.xml`) before reporting a case as passing. The agent's summary is
not the evidence; the file is.

After the agents return, compare both worktrees with their starting snapshots. Clean up changes the review itself created and report them. Leave concurrent contributor edits intact.

## Step 2 — Synthesize

1. **Verify before you report.** Re-read the code for every finding you will
   rank above follow-up. Don't relay an agent's severity, and don't relay a
   finding you haven't sized yourself. If more than about five findings rank
   above follow-up, launch one verifier agent per finding in a single message.
   Tell each one to try to **disprove** its finding (unreachable, already
   handled, older than this PR, or tested elsewhere) and return CONFIRMED,
   DOWNGRADED (with the new severity) or REFUTED. Paste the read-only block into
   these prompts too.
2. **Find where each finding came from.** Did the spec ask for it, did the
   author decide it, or is it older than this PR? A faithful implementation of a defective spec belongs with the product owner. If the implementation matches a design the owner now wants to change, label it as a design change.
3. **Drop what the maintainer already decided**, and say once that you dropped it.
4. **Don't guess authorship.** An AI co-author trailer doesn't tell you which
   lines an agent wrote. When a project requires human-authored changes in an area, ask the author how the work was done.
5. **Rank:**
   - **blocker** — harms customers, data or compliance on merge;
   - **before merge** — a cheap fix the author should make in this PR;
   - **follow-up** — offer to file a ticket;
   - **decision for the maintainer** — product or UX consequences the maintainer has to own.
     Present these; don't decide them.
6. **Cut low-value comments.** Ask "do we really care?" of each one. If a
   finding costs a review round and protects nothing a customer would notice,
   drop it or move it to optional.

## Output

1. **Merge recommendation:** merge / merge after fixes (list them) / hold
   (why). Include sequencing, such as a blocker issue that must merge first.
2. **Findings table**, ranked, with file:line evidence and origin (author, spec
   or pre-existing).
3. **Decisions for the maintainer.** One short paragraph each: the decision, who it
   affects, what it commits us to, and the alternative.
4. **Where human eyes go.** If Agent E and your own check of its artifacts
   already cover something, say so, and don't ask the maintainer to repeat it. If nothing
   needs a human check, say "nothing required" plainly. Otherwise, be specific enough to
   act on in ten minutes:
   - **Code to read yourself:** file:line ranges and the question to hold while
     reading. Pipeline code, permission checks, anything that writes customer
     data, and changes to authoritative product context always qualify.
   - **UI to QA:** name the screens, states, entity types, roles, legacy records, copy to read in place, and the shortest path using the project's test data or preview.
   - **Real-run checks:** identify the relevant end-to-end process, file opened in its intended viewer, or event received in analytics. Verify preview routing and external destinations before proposing a real run. Use a test organization, controlled recipients and a test analytics destination where applicable.
5. **PR comment draft**, in markdown, ready to paste. Open with what was done
   well and is true. Then "Before merge" (numbered), then "Optional". Label
   design changes as the maintainer's. Keep questions for the author as questions. Copy it to the clipboard when asked. Post only on request.
6. **Next step:** after the author pushes fixes, suggest the project's independent final code review on that exact head. If a paid or manually initiated review is available, leave initiation to its owner.

## Re-review (`--since <sha>`)

When the author pushes fixes, fetch and update the worktree (Step 0). Then run
`git merge-base --is-ancestor <sha> HEAD`. If it succeeds, review
`git diff <sha>..HEAD`. If it fails, the author rebased, and that diff would
include everything the base gained since; use
`git range-diff origin/<base> <sha> HEAD` instead. Review the result against
the comments that were posted. For each comment,
say whether it was addressed, partly addressed, or not. Then treat the fix
commits as new code (class G): check them for new problems, not only against
the comment they answer. Re-run the gates on the touched files.
