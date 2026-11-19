---
name: decisions
description: Identify open product, UX, and technical choices before implementation. Use when a builder needs decisions up front or a user wants recommendations on unresolved scope.
---

# Surface the decisions a builder needs

Read the request, prior decisions, applicable project guidance and the code or designs that constrain the answer. First separate facts you can establish yourself from choices a product owner actually needs to make. Trace the current experience where issue text and implementation disagree.

Walk one ordinary journey and one realistic interruption. Look for defaults, permissions, empty states, recovery, ownership of data and consequences for existing users. Check whether an apparent implementation detail quietly changes what someone can do. If existing behavior is awkward, consider keeping it, correcting it, or removing the path entirely. Each has a different cost.

Return a short numbered decision list in dependency order. For each item:

1. Ask one question in language the decision maker uses.
2. Describe the current behavior or evidence that makes it a real choice.
3. Give viable alternatives with the user-visible and engineering consequences.
4. Recommend one and explain why. State which work an answer blocks.

Keep settled decisions, reversible builder choices and factual unknowns out of the voting list. If nothing material is open, explain which evidence settled the apparent ambiguity. Record answered decisions in the requested handoff or draft. Updating a shared tracker requires an instruction to do so.
