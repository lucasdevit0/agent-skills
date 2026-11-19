---
name: cto-review
description: Pre-merge CTO-level review of a PR/branch against its Linear issue — completeness vs scope and session decisions, product/ops/perf gotchas, and a fresh-eyes code pass, synthesized into a merge recommendation. Use when the user asks to "review this PR like a CTO", "check the PR against the Linear issue", "pre-merge review", or "find issues before we merge". Requires a diff to review. NOT for plain code review of a diff (use /code-review), and NOT for reviewing a plan or an unbuilt issue before it's built (use /cto-plan-review).
argument-hint: "PR number or Linear issue ID — requires a diff; for an unbuilt plan use /cto-plan-review"
effort: high
model: claude-fable-5
---

# CTO pre-merge review

**Host tools.** Use the host’s `/code-review` for an independent diff review if available, or request an independent code review of the same commit range.

Review the current PR/branch the way a CTO would before merge: not just "is the code correct" but "did we build what the issue asked for, what breaks or changes in production the moment this merges, and what product decisions got made silently along the way."

## Requires a diff

This skill reviews built work: a PR, a branch ahead of `main`, or uncommitted changes. If there's no diff yet — the issue is still a plan — stop and run `/cto-plan-review` instead, which judges whether it's the right thing to build and whether to greenlight it.

## Step 0 — Gather context

1. **The diff.** `git diff main...HEAD` (or against the PR's base branch; if a PR number was passed, `gh pr diff <n>` / `gh pr view <n>`). Include uncommitted changes (`git diff HEAD`) if present.
2. **The Linear issue.** Resolve the ID from the argument, branch or PR title. Fetch the issue and all comments with the connected Linear tools. Read directly related issues. Comments can carry decisions and guardrails that supersede the issue body.
3. **Session decisions.** If the current conversation contains decisions the user made while building (scope cuts, "keep X as-is", locked choices), collect them — they are scope items too.
4. Build a numbered **scope list**: every concrete deliverable from the issue body + comments + session decisions, and mark which items were explicitly deferred.

## Step 1 — Three parallel review agents

Launch all three in ONE message via the Agent tool (subagent_type: general-purpose). They inherit this turn's model. Give each the repo path, the diff command, and the relevant context inline (paste the scope list — don't make agents re-fetch Linear).

> ### ⚠️ Put this in every agent prompt, verbatim
>
> **The review is READ-ONLY. Do not mutate the working tree or the repo.** No
> `git stash`, `git checkout`, `git switch`, `git reset`, `git restore`, no
> commits, no edits, no `npm install`, no writing files inside the repo.
>
> To compare against another ref, read from it directly — `git show main:path/to/file`
> — or build into a temp directory outside the repo. A stash/pop cycle can lose uncommitted work or reintroduce edits the operator deliberately reverted.
>
> Running the project's own read-only checks (`tsc --noEmit`, `eslint`,
> `prettier --check`, `vitest run`, a CSS compile to a path in `/tmp`) is fine —
> those don't touch tracked files. Anything that writes inside the repo is not.

After the agents return, compare `git status --short` with the starting state. Account for any review-created files. Preserve concurrent user edits and report any unexpected difference.

**Agent A — Completeness auditor.** For every scope item: verify it is ACTUALLY implemented by reading the code (never trust commit messages). Verdict per item: DONE (cite file), PARTIAL (what's missing), MISSING, or DEFERRED-OK. Also list every change in the diff NOT covered by the scope list — surprises a reviewer should know about (new deps, asset weight, hardcoded IDs/secrets-adjacent values, dead code, drive-by changes).

**Agent B — CTO product/ops/perf lens.** Concretely assess:
- **Merge consequences**: what production serves the moment this merges (auto-deploy?), what users see during any interim window, data/dataset state already mutated by scripts run during development.
- **Performance regressions**: new JS shipped to previously-light pages (islands, statically-imported SDKs like analytics libs), unoptimized assets, blocking work.
- **Analytics/SEO correctness**: events that fire with wrong/missing properties, sitemap/robots/canonical drift, consent-gate behavior on opt-in AND opt-out paths.
- **Silent product-decision drift**: places the new implementation quietly differs from the predecessor/spec (a page added to or dropped from the sitemap, changed titles, changed behavior) — flag as decisions for the user, not bugs.
- **Ops readiness**: env vars that must exist in the deploy target, schema/config that must be deployed alongside, anything a runbook needs.

**Agent C — Fresh-eyes code pass.** Correctness review of the hand-written integration code in the diff (skip vendored/ported bulk unless it's wired into new infrastructure). If /code-review already ran this session, list its fixed findings in the prompt and tell the agent NOT to re-report them. Findings need file, line, and a concrete failure scenario, tagged CONFIRMED or PLAUSIBLE.

## Step 2 — Synthesize

1. Dedupe across agents. Personally verify (Read/Grep) any finding you'll call a blocker — never escalate on an agent's word alone.
2. Rank: **blocker** (would harm users/data/SEO on merge) / **should-fix-before-merge** / **follow-up** (file a ticket) / **decision-for-user** (product drift — present, don't decide).
3. **Fix code**: apply clear correctness fixes directly (blockers + cheap should-fixes), re-run the project's check/build, and commit with a descriptive message. Do NOT push without asking unless the user already authorized pushing this branch.
4. **Never silently resolve product decisions** — list them as explicit questions.

## Output

End with a compact report:
- **Merge recommendation**: merge / fix-then-merge (list the fixes, note which you already made) / hold (why).
- Findings table ranked by severity with file:line evidence.
- **Questions for you** — the product-decision drift items.
- Follow-ups worth filing as tickets (offer to create them in Linear).

Offer (don't do unsolicited): post the review summary as a comment on the Linear issue or the PR.
