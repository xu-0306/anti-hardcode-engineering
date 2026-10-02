---
name: anti-complexity-engineering
description: Use when designing, planning, fixing bugs, or reviewing changes that add an entity, status, table, protocol, layer, contract, verifier, or per-request pass, or when a bug fix wraps a layer around existing code. Aligns with reference implementations, maps features onto existing primitives, budgets new concepts, records deletions, and keeps plans proportional. Skip for mechanical edits.
---

# Anti-Complexity Engineering

Use this skill as a design-time guardrail against accidental complexity at the system level: more concepts, layers, and parallel lifecycles than the feature essentially needs. It is the companion of anti-hardcode-engineering: that skill stops fixes that are too narrow, this one stops designs that are too big. For module-level interface design (deep versus shallow modules), use a design lens such as minimise-complexity or codebase-design; this skill works one level up and asks how many things exist and why.

## Core Rule

Every concept must earn its place against a reference and an observed requirement. The default change keeps the concept count flat or shrinks it.

A concept is anything a maintainer must learn: an entity, status, table, config field, protocol, layer, pass that runs on every request, or a new name for an existing thing.

For a bug fix, first ask: would this bug exist if the layer it occurs in did not exist? If not, that layer is the root cause, even when patching around it is the smaller change. See the root-layer statement in section 4.

## 1. Reference Alignment

Before designing, find how at least one mature implementation solves the same problem: a vendored `reference/` directory, a dependency, or a well-known open-source project. Write one line:

> Feature X in <reference> = primitive Y + Z (roughly N concepts, M lines).

Compare it with the proposed design. Every concept beyond the reference needs a requirement the reference does not have. "We might need it" is not a requirement. If no reference exists, say so and keep the first version minimal.

## 2. Primitive Mapping

Name the system's core primitive (for example a request loop, session, job, or document). Express the feature as a composition of existing primitives before inventing anything. Examples from agent runtimes; substitute your domain:

| Feature | Thin shape |
|---|---|
| subagent / delegation | a tool that runs a child loop with fresh context and a tool allowlist, returns text |
| long-running goal | objective, status, and budget attached to an existing session, plus a continuation trigger |
| schedule | stored cron entry that submits a normal request |
| plugin / extension | event hooks plus tool/command registration at load time |
| capability limit | the set of tools actually exposed, not a prediction checked later |

Add a new primitive only when no composition works, and make it replace something rather than sit beside it. Two entities with their own create/resume/cancel/event lifecycle for one idea is the main warning sign.

## 3. Concept Budget

Before implementing, list every concept the change adds:

| New concept | Observed requirement it serves | What it replaces, or why the reference does without it |
|---|---|---|

- No row, or no observed requirement: cut it.
- A status must change behavior somewhere; otherwise it is a log field.
- A config value nobody changes is a constant.
- One implementation behind an interface is a hypothetical seam; two is a real one.
- Default budget for a bug fix: zero new concepts. For a feature: what the reference needs.

## 4. Deletion Ledger

Every change states what it deletes, merges, or makes obsolete, or "nothing, because ...".

- Fix the root cause in place. Do not wrap another guard, verifier, contract, or fallback around a component to catch its failures.
- Root-layer statement, required for every bug fix:
  - Name the layer the bug occurs in and answer the counterfactual: would the bug exist without it? "None" is a valid answer when the bug is in core logic.
  - Classify that layer. A **boundary** limits what can happen: exposed tools, sandbox, permissions, validation at a trust boundary. Keep it and fix it in place. A **derived layer** restates something the system already knows or decides: a prediction then enforced by a gate, a cache of derived state, a translation between two models of the same thing. It is the suspect.
  - When a derived layer is the root cause, a minimal fix now plus a Deferred entry for removing that layer is the expected outcome. The removal need not happen in this change, but keeping the layer without a Deferred entry is incomplete.
- Deletion test: imagine deleting the module. If complexity vanishes, delete it. If it reappears across callers, keep it.
- Chesterton's fence: learn why a layer exists (callers, tests, history) before deleting it. Unknown purpose is a reason to investigate, not to stack more around it.
- Code with no production caller is a deletion candidate, not a foundation.

## 5. Proportional Planning

Plans are complexity too.

- Plan length scales with the change. A one-feature plan with hundreds of lines or many stages is a signal to cut scope, not to add stages.
- No inventory-only stages; inspection belongs inside the first real step.
- Each stage ships something usable or deletes something. Each plan lists its deletions.
- Ship the thin path first, then add what real use demands. Migrate by strangling: build the thin path beside the old one and delete the old one after parity. Avoid both big-bang rewrites and permanent coexistence.
- In multi-agent or multi-session work, hand off the concept budget and deletion ledger so the next agent does not add layers blindly.

## Legitimate Complexity

Do not simplify these away, even when the reference is thinner. Keep them, but keep each in one place:

- security boundaries: authorization, sandboxing, approvals, secret handling, validation at trust boundaries
- data-loss prevention: transactions, undo, backups, migrations
- concurrency control for a race that actually occurred or is inherent in the design
- audit or compliance that is actually required
- performance work backed by a measurement

## Precedence Over Other Skills

Skills that prescribe adding structure (verification pipelines, contracts, obligation tracking, extra gates) apply at the boundary where their problem was observed, not globally. When they conflict with this budget, add only what the specific observed failure needs and record it as a budget row. A guardrail applied everywhere becomes the architecture.

## Red Flags

- a hub file or class that every feature edits
- the system re-implementing its own platform: a second plugin loader, scheduler, workflow engine, or task model beside an existing one
- predict-then-gate: something predicts what must happen, a gate enforces the prediction, and failures come from wrong predictions
- several entities with near-identical lifecycles for one idea
- layering vocabulary (contract, verifier, fingerprint, projection, coordinator, lease) growing faster than features
- one solution style applied to every problem
- bug fixes that always add lines and almost never remove any

## Audit Mode

When asked whether a codebase is over-engineered, measure before judging:

1. hubs: largest files and classes by lines and method count
2. concepts per feature: entities, tables, statuses, request-model fields
3. vocabulary: counts of layering words such as contract, verifier, manager, coordinator, policy, lease
4. dead weight: modules imported only by tests or evaluation code
5. reference comparison: the same feature's concept count and size in a reference

Size alone is not the verdict; references can be large too. Compare concepts per feature. Report essential versus accidental complexity, the top three delete-or-merge candidates, and what to freeze until they land.

## Review Checklist

Reject or revise a change when any item is true:

- It adds a concept without a budget row.
- It wraps a layer around a component instead of fixing the component.
- It fixes a bug without the root-layer statement, keeps a derived root-cause layer without a Deferred removal entry, or removes or weakens a boundary to make a bug go away.
- It creates a second entity or lifecycle for an existing idea.
- It adds a per-request pass (classifier, verifier, contract) for a failure observed once, at one place.
- It deletes nothing and does not say why.
- It designs a feature mature projects already ship without checking any of them.
- Its plan has more stages than the change needs.

Approve when:

- The reference alignment line is present and differences are justified.
- The feature is mapped onto existing primitives.
- The concept count is flat or down, or each addition has a budget row.
- The deletion ledger is present.
- Legitimate complexity is kept and localized.

## Response Pattern

When reporting a relevant design, plan, or fix, include a concise note. If the user requires a specific output format, put each item inside its closest section of that format (for example root layer and deferred removals under deletions) rather than dropping it.

- Reference: how <reference> does it, in one line
- Primitive mapping: the composition used
- Concept budget: +N / -M, with the added concepts named
- Deletion ledger: what was removed or merged, or "nothing, because ..."
- Root layer (bug fixes): the layer, boundary or derived, and whether the bug would exist without it
- Deferred: what was intentionally not built or removed, including any kept suspect layer, and what would trigger the change
