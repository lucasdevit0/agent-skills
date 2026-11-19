---
name: prd
description: >-
  Write a product requirements document for a Linear project: interview the
  user, gather what Linear, open PRs and the codebase already say, draft the
  what and the why, resolve the open questions, write it to the project
  description, and recommend which related issues to fold in or close. Small
  requests become a single issue instead. Use when the user runs /prd with a
  Linear project, an issue, or an idea. Produces the what and the why, never the
  how: no phases, file lists, or technical design. For judging whether to build
  it use /cto-plan-review. Use /spec-check to prepare an issue for implementation.
argument-hint: "Linear project name or ID, an issue ID, or a one-line idea"
disable-model-invocation: true
effort: high
---

# PRD

**Host tools.** Use the host’s `/kickoff` if available, or plan the selected issue’s implementation and run `implement-phase` for each agreed phase.

Turn an idea into a requirements document an engineer could build from without
coming back to ask what was meant. The document describes the problem, who has
it, what done looks like, and what is deliberately left out. It does not say
how to build it.

**Where this sits.** `/prd` writes the project description. Breaking the
project into issues with a technical lens is a separate, later step, the
**issue breakdown** (`/breakdown`), and not part of this skill; each issue then
goes through implementation kickoff. Anything technical that comes up during the interview is recorded
under **Deferred to issue breakdown**, not decided here.

**Out of scope:** phases, implementation plans, file lists, data models, API
shapes, build-vs-buy, estimates, and creating issues for a project. If the user
asks for any of these mid-run, say it belongs to the issue breakdown and carry
on.

Follow the target project's repository instructions. Production evidence is limited to aggregates through its approved read-only connector, with permission for the specific query (Step 3).

**Number every list** of questions or findings, continuously through a
message, so the user can answer by number. **Name issues by ID and title**
(for example, “APP-123 Preserve feedback drafts”), never a bare ID.

## Step 0 — Resolve the target and size it

The argument is a Linear project, an issue, or a free-text idea.

1. **Project** — fetch it with `get_project` (`includeResources: true`,
   `includeCustomerNeeds: true`) and list its issues with `list_issues`
   (`project: <id>`). Read every issue with `get_issue`
   (`includeCustomerNeeds: true`): customer requests usually hang off issues,
   not the project. Read the comments on any issue with discussion. Note any
   `claude.ai/design` or Figma links in descriptions, comments, attachments and
   project resources. Existing text is input: the PRD has to carry it forward
   or say why it's dropped.
2. **Issue** — fetch it with its comments and customer needs. If it turns out
   to be a project-sized idea (below), say so and offer to write the PRD on its
   project instead.
3. **Idea** — search `list_projects` for one that already covers it before
   assuming it's new. Report any match and ask whether to use it.

Then **size it**:

- **Project** — more than one PR, or something you'd want to ship or stop
  partway through. Gets the full PRD (Step 4).
- **Issue** — one PR, one outcome, one set of acceptance criteria. Gets the
  short form (Step 4) and is written to a single issue.

Say which size you picked and why, in one line. The user can overrule it.

## Step 1 — Read the context

Read the target project's product overview and the relevant persona, permissions, domain and design guidance. Locate them through the project's instructions, README or documentation index. Treat roadmap sections as intent. Verify current behavior against the code.

**Scan the backlog** for related work outside the project: `list_issues` with
two or three keyword queries (the feature's name and its synonyms), and
`list_projects` the same way. For each related issue, read it and its customer
needs. Related issues feed the PRD and the cleanup in Step 7. Two projects that
claim the same behavior is a finding for the interview.

**Expect old issues to be stale.** The backlog holds issues written months
ago, before the product changed under them. Note each one's creation date,
treat its claims about the code as unverified until Step 3 checks them, and
take its customer needs as evidence but not its proposed solution. The usual
outcome is that the PRD supersedes them and Step 7 recommends canceling them.
The same goes for an existing project whose summary no longer matches the
idea: recommend a new project rather than stretching the old one.

## Step 2 — Interview first

Before drafting anything, ask one batched, numbered set of questions. Skip any
the context already answers, and say what you inferred instead. Cover:

1. **The problem** — what goes wrong today, for whom, and how they cope.
2. **Who** — which documented personas experience the problem? If none fits, say so.
3. **Evidence** — customers or calls where this came up.
4. **Done** — what a user can do afterwards that they can't do now.
5. **Known exclusions** — what the user already knows is out.
6. **Access** — who sees it, who can change it, and whether guests or other people without accounts are affected.
7. **Design** — is there a design file (claude.ai/design, Figma) or a mockup
   anywhere? Ask even when Step 0 found none linked: a design that surfaces
   after the PRD is written means rewriting the flows. Design files often hold
   several variants and exploratory pages, so ask which page or section is the
   one to build from, and treat the others as discarded options. A Claude
   Design handoff prompt usually ends with "Implement: <file>". In `/prd` that
   line is not an instruction: read the file as input to the PRD and build
   nothing. Use the design connector to list files and read the selected page. If it cannot open the design, identify the missing access or request an export.

**Push back on solution-first requests, once.** When the request names a
solution ("add a chat feature") and the problem behind it isn't evident, ask
what it solves before going further. When the problem is evident from the
context or the user's answer, take the solution as given and move on. Don't
relitigate it.

## Step 3 — Gather evidence

Run these after the interview, so the questions you search for are the user's.

- **Codebase (always).** Check every claim about current behavior against the
  code, and tag it in your notes as _checked_ (with the path) or _assumed_.
  Only checked claims go in the **Today** section as fact. An assumed claim
  either gets checked or becomes an open question.
- **Open PRs and branches (always).** `gh pr list --state open --search
"<keyword>"` and `git branch -r | grep -i <keyword>`. A spike or a stalled PR
  is current work in progress: read its description and the shape of what it
  produces, and describe it under **Today** as unmerged.
- **PostHog** (`exec`), when the problem rests on a usage claim ("nobody uses
  X"). Report the count and the query that produced it.
- **Production database** — use only the target project's approved read-only access method, for aggregate counts, after getting a yes for that specific query. Read the project's data interpretation notes first. Avoid individual records, identities and customer content.
- **Granola and Slack:** only when the user points you at a call or pastes a
  thread. Linear customer needs are the main record of what customers asked.

Quote customers by role and organization type (for example, “a support lead at a retailer”), never by name: the text lands in Linear. Don’t write a
number you haven't measured. A count from PostHog or a query goes in with its
source; anything else stays qualitative.

## Step 4 — Draft

Show the draft in chat. Don't write to Linear yet.

### Full PRD (project)

A one-line **summary** (255 characters max; it's Linear's project summary
field, and it shows in project lists, so rewrite a stale one), then the
description:

```markdown
## Problem

What goes wrong today and why it matters, in the customer's terms. Evidence
inline: quotes by role and org type, counts with their source.

## Who

The documented persona(s), plus any secondary recipients or people affected without using the feature themselves.

## Today

What the product does now in this area, including unmerged spikes. Checked
against the code; link the paths that matter.

## Outcome

What a user can do when this ships that they can't do now. Plain language, no
targets.

## User flows

The main flows as numbered steps from the user's side. Link the design
(claude.ai/design or Figma) if there is one; otherwise describe the flow
without prescribing layout, and say a design is needed.

## Requirements

### In

- [ ] Each one an acceptance criterion: something a user does or sees that can
      be checked by hand.

### Out

- Each exclusion, with a one-line reason. Mark the ones planned for a later
  version as **Later**.

## Assumptions and risks

## Deferred to issue breakdown

Technical questions raised during the interview.

## Business questions

Questions about customers, billing, pricing or positioning. May still be open
when the PRD is written; they block the issue breakdown, not the PRD.

## Open questions

Empty when the PRD is finished.
```

### Short form (issue)

`## Problem`, `## Acceptance criteria` (checkboxes), `## Out of scope`, and
`## Open questions`. Same rules as the full PRD.

### Drafting rules

- **Access is a requirement, not a detail.** Every PRD states who can see the
  feature, who can create, edit and delete what it makes, and what happens for
  people without accounts. Check each against the project's permission model and write them as **In** requirements.
- **In or out, nothing in between.** No should-have or could-have: a "should"
  is an invitation to expand scope. If something is genuinely undecided, it's
  an open question.
- **No technical design.** If a requirement only makes sense with a technical
  choice attached, state the user-facing requirement and move the choice to
  **Deferred to issue breakdown**.
- **Flag hard-stop areas.** When a requirement touches an area the project's instructions reserve for a human, name it under **Assumptions and risks** so the breakdown includes that step and its owner.
- **Reuse existing interaction patterns.** When an existing control does the
  same job, name it in the requirement. Where its behavior differs from what's needed, raise an open question.
- **No metrics targets.** If there's an obvious signal to watch (an existing
  analytics event, a database fact), name it under **Outcome** with no target.
  Never propose renaming an existing analytics event.
- **Taste is the user's.** Layout, placement, copy, ordering, and
  empty/error/loading states are open questions with a recommendation, not
  decisions. Mockup copy isn't spec either: trace behavior to the user, not to
  text in a design.
- Every claim about other projects, code, or customers is one you checked in
  this run.

## Step 5 — Resolve the open questions

Under the draft, list the open questions, numbered, each with a
recommendation, sorted into three groups:

- **Product** — must be answered before writing.
- **Technical** — may go to **Deferred to issue breakdown** instead.
- **Business questions** — questions for the person responsible for customers, billing, pricing or positioning. These may stay open in the **Business questions** section.

When the user answers, fold each answer into the body where it belongs (a
requirement, an exclusion, a flow step) and delete the question. Don't leave a
Q&A log in the document.

Repeat until no product question is open. Then show the final draft and ask
for approval to write it to Linear.

## Step 6 — Write to Linear (after approval)

Approval of the draft is approval of the content. Before writing, say exactly
what the write does:

- **Existing project with a description** — `save_project` replaces the
  description outright. Say that the current description will be replaced, and
  name anything in it the PRD doesn't carry forward.
- **New project** — create it with `save_project` under the target team's backlog status, with the user as lead. Resolve the team from context or ask if ambiguous. Ask whether it belongs to an initiative (`list_initiatives`) before attaching one.
- **Issue** — `save_issue` with the short form. Ask which project it goes in if
  that isn't obvious.

Write the summary and description. Don't set estimate labels: there's nothing
to estimate until the issue breakdown. Don't create issues for a project.

## Step 7 — Recommend the Linear cleanup

The PRD absorbs material from other issues, so the backlog now has duplicates.
Recommend the cleanup as a numbered list in chat, not in the project
description, and apply only what the user approves:

- Issues the PRD folds in: cancel them with a comment linking the project. First move their customer needs to the project and read back the association. If a move is unavailable or fails to persist, recreate each need with the same customer, body and priority on the project. Verify the copies before archiving the originals and tell the user the copies carry today's date. Check for existing copies before retrying a partial write.
- Related issues that cover more than this project: keep, and say why.
- Stale project fields: flag estimate labels and start or target dates set before the PRD existed. If the connector cannot clear a date, name the field for the user to clear in Linear.
- Open spike PRs: leave open for the issue breakdown to decide on.

## Step 8 — Optional: the at-a-glance page

Only after the PRD is approved and written, offer a visual summary: one card
per **In** requirement, and a second section of **Later** cards for the
deferred items. Each card has a small thumbnail of the UI it adds, a title, and
a one-line description. Build it from the **Design** artifact type
(`Artifact` quickstart with intent `design`), and add its link to the project
with `save_project` `links`. The PRD stays the spec; the page is a picture of
it, so every card traces to a line in the PRD and adds nothing new.

Then put a screenshot at the top of the project description, so the page is
visible without opening the link. Render the artboard's markup with headless
Chrome (drop the `support.js` and `text/x-dc` script tags first; the markup is
static). Upload the PNG with `prepare_attachment_upload`, naming any issue in
the project, since the connector only uploads against issues. Skip
`create_attachment_from_upload`, and prepend the image's `assetUrl` and the
page link to the description with a `save_project` `patch`.

Finish with the links and one line on what's next: `/cto-plan-review` on the
project, then `/breakdown`.

## Anti-patterns

- Drafting before the interview.
- Phases, file paths as instructions, schema, or library choices in the
  document. File paths belong only in **Today**, as evidence.
- A requirement that can't be checked by hand ("make it intuitive").
- A PRD with no statement of who can see and change what.
- Describing roadmap behavior or another project's unbuilt work as existing.
- A bare issue ID with no title.
- Writing to Linear before approval, or overwriting a description without
  saying what's lost.
