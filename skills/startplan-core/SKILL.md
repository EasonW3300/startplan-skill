---
name: startplan-core
description: Use when the startplan router delegates a request that needs a grounded, decision-complete plan and a protected approval handoff.
---

# Startplan Core

Plan the request carried by the latest `/startplan` user message. Keep the user's scope and wording intact. The main agent performs this work; do not delegate planning to a built-in plan subagent.

## Activate the planning guard

Before task-specific inspection or questions, call `Workflow` with `action=select_route` and `route=plan`. The Workflow state and its `plan-readonly` allowlist are the hard guard on OhMyAgent. Submit concrete, non-empty evidence with every `complete_step` call.

If the environment already reports permission mode `plan`, keep it. Otherwise, if `EnterPlanMode` is directly available, invoke it after the Workflow starts. If it is not visible, make at most one `ToolSearch` attempt for it; do not repeat discovery when the host does not provide it. Workflow protection remains sufficient for the tools governed by its allowlist.

## Planning loop

Use these reasoning phases inside the `planning` Workflow step. They are a thinking sequence, not separate Workflow steps.

### Ground

Read the relevant environment before questioning the user. For coding work, locate concrete files, symbols, interfaces, tests, configuration, and repository rules with read-only tools. For general work, inspect the provided documents, current context, public sources, or explicitly available read-only resources. Separate observed facts from assumptions and name evidence in the plan when it affects execution.

Do not replace discoverable facts with questions. If a required source cannot be reached with an allowed read-only tool, record that limit and ask only when it blocks a material decision.

### Intent

Resolve the outcome, success conditions, scope, audience, constraints, and user-owned tradeoffs. Ask only about choices that cannot be discovered and would materially change the plan. Prefer one focused round of one to three structured questions; continue grounding while waiting when useful work remains.

### Implementation

Work out the execution path deeply enough that another capable agent can carry it out without making hidden product, architecture, policy, or delivery decisions. Identify concrete change locations or work products, ordering and dependencies, interfaces or handoffs, failure handling, and meaningful verification. Include migration, rollout, rollback, ownership, or external approval only when the request needs them.

## Produce the plan

Write one complete replacement plan rather than incremental notes. Use exactly one `<proposed_plan>` block with an adaptive 3–5 section Markdown structure. Choose headings that fit the request. A useful plan normally covers the objective and grounded context, concrete implementation steps, and verification; add boundaries, risks, delivery, or recovery only when material.

Keep steps specific and outcome-oriented. Name files, modules, systems, artifacts, or responsible roles when known. Do not include placeholders, unresolved high-impact choices, approval hashes or identifiers, or a rigid universal schema.

Immediately before returning the plan, call `Workflow(action=complete_step, outcome=plan_ready, evidence=...)`. This completes the turn-bounded Workflow so OhMyAgent can end the response instead of continuing autonomously. After the transition succeeds, the response must contain only the `<proposed_plan>` block. Do not start implementation in the same turn.

## Handle the next user turn

The planning Workflow is complete after the plan is presented. Use the latest plan and the user's direct reply as the handoff state.

- **Execute:** Accept “按当前计划执行” or an equally unambiguous direct instruction to execute the latest plan. If native permission mode `plan` is active, invoke `ExitPlanMode` first. Then begin executing that plan.
- **Revise:** When the user changes requirements or requests plan edits, start a fresh protected planning turn with `Workflow(action=select_route, route=plan)` before any new inspection. Return to Ground, Intent, and Implementation as needed, then present a complete replacement plan.
- **Cancel:** When the user cancels, exit native plan mode if it was active, then stop without execution.

Replies such as “继续”, “可以”, or “行” are not execution instructions. Ask for an unambiguous instruction if the user intends execution, and perform no implementation in that turn.
