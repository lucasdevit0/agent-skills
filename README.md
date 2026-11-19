# Agent Skills

These are the workflows I use when I build applications with coding agents. I want to make the product decisions myself and give the agent enough context to implement them, test the result, and show me what still needs my attention.

## How I use them

Say I want people to submit feedback from inside an app. I start with `prd` to work out what someone should see after they hit **Send**. If delivery can fail, `decisions` helps me settle whether their draft survives and how they retry. Once that's clear, I can hand a phase to `implement-phase` with a concrete acceptance criterion: someone can retry a failed submission without typing their feedback again.

Before I merge, I want to see that path exercised and understand what happens to submissions already in progress when the change ships. That's where `phase-check` and `cto-review` come in. For a small fix I can go straight to the skill I need.

| Skill | When I reach for it |
| --- | --- |
| [prd](skills/prd/SKILL.md) | I have an idea and need a clear user outcome, scope, and acceptance criteria. |
| [decisions](skills/decisions/SKILL.md) | I want the open choices and a recommendation for each before work starts. |
| [two-hats](skills/two-hats/SKILL.md) | One decision looks different from the engineering and product sides. |
| [three-hats](skills/three-hats/SKILL.md) | The way someone experiences the interface could change that decision. |
| [cto-plan-review](skills/cto-plan-review/SKILL.md) | I want a second opinion on whether to build something and in what order. |
| [breakdown](skills/breakdown/SKILL.md) | The requirements are settled and I need phases someone can pick up independently. |
| [spec-check](skills/spec-check/SKILL.md) | I'm about to hand off a ticket and want its assumptions checked against the code. |
| [implement-phase](skills/implement-phase/SKILL.md) | I'm ready to build and verify one agreed piece of work. |
| [phase-check](skills/phase-check/SKILL.md) | I want to catch behavior lost in my own diff before handing it to a reviewer. |
| [cto-review](skills/cto-review/SKILL.md) | I need a merge decision that accounts for scope, production behavior, and rollback. |
| [pr-review](skills/pr-review/SKILL.md) | I'm reviewing someone else's PR and want specific feedback and a short QA list. |
| [implementation-explainer](skills/implementation-explainer/SKILL.md) | I want a visual walkthrough of what shipped and how it works. |
| [integration-nextjs-app-router](skills/integration-nextjs-app-router/SKILL.md) | I'm using PostHog with Next.js App Router and want to verify a real user journey. |

## Use a skill in another project

Each folder under [skills](skills) is self-contained. Copy the whole folder for the skill you want into one of these locations, or point your agent directly at its downloaded `SKILL.md`.

| Agent | For one project | For all your projects |
| --- | --- | --- |
| [Codex](https://developers.openai.com/codex/skills/) | `.agents/skills/` | `~/.agents/skills/` |
| [Claude Code](https://code.claude.com/docs/en/skills) | `.claude/skills/` | `~/.claude/skills/` |

Give it the context for the job. For example: **“Use spec-check on this feedback ticket. Check the retry behavior against the current form and tell me what a developer would still have to guess.”** The target project supplies its own commands, conventions, and release process.
