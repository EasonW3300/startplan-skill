---
name: startplan
description: Use when a user invokes /startplan followed by a request and wants a grounded plan before implementation or external action.
---

# Startplan Router

Treat the text after `/startplan` in the latest user message as the complete planning request. Preserve its meaning and boundaries.

Immediately invoke the `Skill` tool for `startplan-core`. Do this before inspecting the task, asking questions, drafting a plan, or taking any external action. After the core skill loads, follow it for the rest of the planning session.

## Prompt-only fallback

If the `Skill` tool or `startplan-core` is unavailable after one attempt, say that this fallback is enforced by instructions and is not hard read-only protection. Then:

1. Ground the request in available read-only evidence.
2. Resolve only material intent choices that cannot be discovered.
3. Work out implementation details and verification.
4. Return one complete `<proposed_plan>` block and stop.

Do not write, edit, execute mutating commands, send messages, schedule work, or perform other external actions during fallback planning. Require a later, unambiguous instruction to execute the current plan; replies such as “继续”, “可以”, or “行” do not authorize execution.
