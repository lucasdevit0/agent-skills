---
name: implementation-explainer
description: Explain implemented changes in plain language through a local visual guide grounded in the actual code. Use when the user requests an implementation walkthrough, technical explanation, or an explanatory document after development work.
---

# Implementation Explainer

Create a clear, detailed explanation of what was implemented, how it works,
and why the important technical decisions were made.

Write for a product owner who understands the product and wants to understand
the engineering without having to read the code. Use simple language without
oversimplifying behavior or hiding limitations.

## Establish the scope and evidence

Use the current conversation and the requested issues, feature, commits, or
branch to identify what to explain. If the scope is clear, proceed without
asking the user to repeat it.

Read the relevant repository instructions, product context, implementation,
and tests. Use issue descriptions and design files when they help explain
intent, but treat the actual code as the source for current behavior.

Distinguish:

- What is implemented.
- What was tested, and how.
- What still requires deployment, configuration, or a migration.
- What remains planned, incomplete, or uncertain.

Do not present an intended behavior as implemented merely because it appears
in a plan. Do not invent the rationale behind a decision. When the rationale
is inferred from the code, say so.

Identify the branch and commit or working-tree state being explained so the
document is a dated snapshot rather than an evergreen claim.

## Explain from the user's perspective

Start with a short explanation of the problem and the resulting experience.

Walk through a realistic example, following the user's actions and explaining
what happens behind the scenes at each important step.

Then explain the relevant technical details. Depending on the feature, cover:

- Where the work happens: browser, server, database, background job,
  or external service.
- What information enters the system, how it changes, and what comes out.
- What is saved, where it is saved, what is temporary, and what is deleted.
- The important buttons and state changes, including what counts as
  completion or success.
- Authentication, permissions, limits, and other meaningful safeguards.
- Errors, timeouts, retries, cancellation, and recovery.
- Analytics events: what triggers each event, when it fires, and what
  properties it contains.
- Migrations, environment variables, deployment steps, and dependencies.
- Important tradeoffs, accepted limitations, and unfinished work.

Include only topics relevant to the implementation. Do not force every
document into the same checklist.

Explain technical terms when first used. Describe a component's role before
introducing its filename or function name.

## Explain AI and structured data when relevant

Explain what the AI is responsible for and what deterministic application
code is responsible for.

Include the actual prompt, or clearly identify an excerpt. Explain its key
instructions in plain language. Distinguish fixed prompt text from runtime
inputs.

Show a representative JSON example consistent with the implemented schema.
Use invented example data and label it as an example.

Explain:

- What the important fields mean.
- Required, optional, and nullable fields.
- Allowed values where they affect behavior.
- How responses are validated and transformed.
- What happens when a response is invalid or incomplete.
- Whether missing information is omitted, defaulted, or inferred.

Separate AI output JSON, editor JSON, API responses, and database records
when they are different structures. Do not present them as interchangeable.

Include the model and relevant limits only when verified from the
implementation. Do not infer provider retention guarantees from an
application setting such as store: false.

## Use visuals to improve understanding

Use diagrams when they explain something more clearly than prose:

- A flow diagram for the journey from input to output.
- A sequence diagram for interactions between services.
- A state diagram for progress, completion, and recovery.
- A before-and-after comparison for a behavior change.
- An annotated example for a prompt, JSON structure, or screen.

Label diagrams with understandable names and make their arrows meaningful.
Keep technical identifiers secondary.

Use real screenshots when explaining implemented UI. Clearly distinguish
screenshots, design references, and illustrative diagrams.

Do not add decorative diagrams or require a fixed number of visuals.

## Produce the local guide

Default to a self-contained HTML document that opens locally in a browser
and works offline. Use embedded styles and inline SVG for diagrams; avoid
remote fonts, CDNs, or external rendering dependencies.

Make it readable on desktop and mobile, with sensible print styling.
Use expandable sections for long prompts, schemas, and deeper technical
details so the main explanation remains approachable.

Save the document outside the repository, in an authorized artifact/output
directory, unless the user explicitly chooses another location.

Use a descriptive filename. Do not overwrite an unrelated document.

Include a compact source section pointing to the important implementation
files and verified issue links. Avoid an exhaustive file inventory.

This skill explains the implementation. It does not authorize code changes,
commits, deployments, migrations, external updates, or messages.

## Verify and hand off

Check the explanation against the code, especially claims about saved data,
permissions, completion, retries, and failure handling.

Reuse relevant verification results already available. Distinguish checks
performed now from earlier results; do not rerun expensive or state-changing
work merely to write the guide.

Open the generated document and inspect its rendering. Check diagrams,
examples, disclosure controls, and embedded assets.

Return a clickable link with a short description of what the guide covers
and any important uncertainty.
