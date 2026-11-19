---
name: phase-check
description: Self-review a just-written phase or fix before handing it to a reviewer. Use to catch behavior displaced by a change and weaknesses in the author's own verification.
---

# Review your own diff

Identify the baseline and include staged, unstaged and relevant new files, plus any fix commits within the phase. Read the acceptance criteria and list what your implementation claims to preserve. Concentrate on the places your own approach makes easy to overlook.

1. **Account for displaced behavior.** When replacing a component, handler or storage path, enumerate what the old version did. For each responsibility, point to its new owner, its explicit removal decision, or a gap. Include defaults, keyboard paths, permission checks, cleanup and error handling when relevant.
2. **Challenge your evidence.** Reread screenshots, responses and logs as a skeptical user. Ask what else could produce the observed result. Check the property your claim actually depends on.
3. **Find alternate forms.** Search for equivalent entry points, old field names, wrapper functions, indirect callers, exports and previously persisted values. Follow the code after the search.
4. **Break the test's assumptions.** Would it fail if the original bug were reintroduced? Check whether the test creates a state the real application never produces or stubs the behavior it claims to prove.
5. **Read the whole experience.** Read surrounding copy in order and follow labels to their destinations. In a nonvisual flow, compare the promised result with the actual response, exit status or saved state.
6. **Review the fixes.** Inspect changes made in response to feedback with the same care as the initial implementation.

Report each material issue with a trigger, user impact, location and the smallest credible correction. State which checks you performed and which claims remain unverified. In a review-only request, return findings. In an implementation phase, fix in-scope findings and repeat the affected checks.
