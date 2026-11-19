---
name: implementation-explainer
description: Turn implemented changes into a plain-language visual walkthrough grounded in the actual code. Use for a feature handoff or an explanation of how a release works.
---

# Explain an implementation

Identify the requested feature and the branch, commit or working state being explained. Read the implementation, its real callers, relevant tests and existing verification results. Use requirements and design files for intent. Distinguish behavior verified in code, behavior observed in use, deployment still required and work still planned. Label any rationale inferred from the code.

## Walk a person through it

Start with the user's problem. Follow one realistic example from first action to visible result. Introduce a component by its job before giving its technical name. Follow the input across the relevant boundaries. Explain what is stored, who can see it, what happens on failure and what a retry would do.

Choose deeper details according to the feature. For an AI workflow, distinguish the model's responsibility from deterministic code. Show a representative, invented input and output, explain validation and fallback behavior, and distinguish generated output from the saved record. Include relevant prompt excerpts only when suitable for the intended audience.

## Produce the guide

Default to a self-contained local HTML guide for a requested visual walkthrough. Use a diagram to answer a specific question, such as who owns a job while a user is offline. Use real screenshots for implemented UI when available and label illustrations. Keep long schemas or prompt excerpts collapsible. Include sources and the revision explained.

Respect a requested format or output location. Otherwise use an authorized artifact directory. Check the rendered guide in a browser, including narrow-screen layout, disclosures, embedded assets and offline readability. Hand over the guide with any material uncertainty about the implementation.
