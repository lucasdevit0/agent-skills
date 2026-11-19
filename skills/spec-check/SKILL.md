---
name: spec-check
description: Audit an implementation brief against the actual project before work starts. Use to find ambiguous acceptance criteria, missing entry points, and unverified assumptions.
---

# Check whether a spec is ready to build

Read the full brief, amendments, related designs and applicable project rules. Convert the proposed behavior into observable outcomes, then inspect the current implementation. For every factual premise that drives work, find evidence and test the conclusion drawn from it. For example, a shared type accepting a new value does not establish that every renderer can display it.

## Follow the change through the project

- Enumerate ways to reach the behavior, including alternate clients, scheduled tasks and direct API calls where relevant. Trace shared code to each affected consumer.
- Check who owns identity, who may act and where the permission is enforced. Distinguish hiding a control from enforcing access.
- Follow failures, retries, repeated clicks, navigation and stale responses. Identify where errors would still be visible after a transition.
- Trace destructive steps around the success condition. What remains recoverable if the next operation fails? What happens to older records and callers?
- Inspect the intended test at the boundary where the behavior occurs. A mocked success may establish very little about the reported bug.
- Check whether the implementer can start with the available setup, fixtures and access. Replace brittle references with stable names or behavior when helpful.

Apply checks relevant to the change. For each gap give the affected flow, a concrete failing scenario, evidence, and proposed wording for the acceptance criterion. Separate questions requiring a product decision from details the builder can determine. Conclude ready, ready after named edits, or blocked on a specific decision. Return the amendments in the requested format and update the tracker only when that write is in scope.
