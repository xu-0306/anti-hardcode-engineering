---
name: anti-hardcode-engineering
description: Use before and during coding when designing, implementing, or reviewing bug fixes, features, routing logic, intent detection, parser behavior, agent/tool runtime behavior, policy checks, validation gates, UI extraction, scraping, or any change where an agent may be tempted to solve the task with hardcoded keywords, language-specific lists, brittle selectors, copied examples, provider strings, magic paths, or one-off special cases. Apply this skill to classify closed-set versus open-world problems, choose abstraction boundaries such as obligations, capabilities, contracts, parsers, verifiers, registries, or adapters, and require tests that cannot pass by merely hardcoding the observed example.
---

# Anti-Hardcode Engineering

Use this skill as an implementation-time engineering guardrail, not only as a review checklist. The goal is to prevent brittle hardcoded designs before they enter the codebase and to make behavior generalize across unseen inputs, languages, providers, paths, and user phrasing when the problem space is open-ended.

## Core Rule

Do not solve an open-world problem with a closed list unless the list is only a fallback, seed, compatibility shim, or telemetry aid.

Prefer durable contracts:

- intent or obligation extraction over keyword matching
- capability exposure over example-specific tool forcing
- parser/grammar boundaries over substring checks
- semantic validation over literal output matching
- completion verification over trusting final text
- structural anchors over brittle selectors
- configuration or registry data over scattered conditionals

## First Classification

Before editing code, classify the problem:

| Problem Type | Examples | Acceptable Fix Shape |
|---|---|---|
| Closed set | official enum, protocol status, MIME type, known model id, fixed API field | explicit mapping, exhaustive switch, table with tests |
| Open set | human intent, natural language phrasing, file/task requests, UI text, provider output variants, webpage layouts | contract, parser, classifier, verifier, capability model, schema, fallback layers |
| Boundary case | legacy compatibility, migration alias, known provider quirk | isolated adapter with comment, expiry plan, regression test |

If the user request, bug report, or failing test involves natural language, model behavior, agent planning, tool selection, artifact creation, scraping, parsing provider text, or user intent, assume open set until proven otherwise.

## When NOT To Use

This skill fixes a specific open-world bug at one boundary. It is not an architecture. Do not apply it, or scale it back, when:

- The change would install the obligation pipeline (derive obligation -> expose capability -> track -> block before final) as a global layer on every turn, request, or subagent. Use it only at the single boundary where the observed bug lives.
- Capability can be expressed by which tools are actually exposed. A read-only worker simply has no write tool; do not let a model predict obligations and then gate them. Predict-then-gate layers create the failures they guard against.
- A reference implementation or an existing primitive (agent loop, tool, session, config) already covers the behavior. Check how mature projects solve it before adding a contract, verifier, or entity.
- The risk is hypothetical. Do not build validation layers for failures that have not occurred and cannot be tested.
- The fix would add a new per-turn semantic pass, new entity, new status, or new table. Prefer deleting or merging an existing layer; if something must be added, state what it replaces.
- The problem is genuinely closed-set or trivial. An explicit mapping or one-line fix is the right answer; do not upgrade it into a classifier.

## Pre-Implementation Gate

Before implementing, answer these internally and let them guide the edit:

1. What invariant should hold beyond the observed example?
2. Is the proposed fix adding a literal token, phrase, path, provider name, or shape copied from the bug report?
3. Would the fix still work for another language, synonym, provider, path, or equivalent UI phrasing?
4. Is there a stronger abstraction already present in the codebase: schema, parser, router, policy, capability, obligation, state machine, or verifier?
5. What negative test would fail if the fix only hardcoded the current example?

If answers are unclear, inspect neighboring architecture before editing.

## Implementation Guidance

### For Intent And Agent Runtime Bugs

At the boundary where the bug lives (not as a per-turn layer; see When NOT To Use), prefer a pipeline like:

1. derive task obligation from the request and context
2. expose tools/capabilities required to satisfy the obligation
3. track whether the required state-changing event occurred
4. block or recover before final answer if the obligation is unmet
5. record telemetry/reason codes for fallback behavior

Do not make language keywords the source of truth. Keyword lists may help recall, but the runtime contract must be completion-based: if the user asked for an artifact, the run must produce or modify the artifact before claiming success.

### For Parser And Tool-Call Bugs

Prefer grammar-aware or structured parsing:

- parse known tags or schemas as a language, not by scattered substrings
- support provider variants at adapter boundaries
- remove consumed control markup from visible text
- preserve non-control reasoning/content when safe
- test malformed, partial, duplicated, and embedded cases

### For Security And Policy Bugs

Prefer policy objects and normalized command/path models:

- normalize before classifying
- evaluate deny rules before allow rules unless policy says otherwise
- keep shell-specific parsing behind adapters
- test equivalent representations, not only the reported command

### For UI, Scraping, And Extraction Bugs

Prefer structural contracts:

- data attributes, ARIA roles, schema fields, stable labels, or DOM relationships
- validation for empty/missing/duplicated output
- layout-independent assertions where possible

Avoid nth-child chains, exact copy strings, and single-page assumptions unless the source is fixed by contract.

## Acceptable Hardcoding

Hardcoding is acceptable when at least one is true:

- the values are an official finite protocol or API contract
- the values are project configuration centralized in one registry
- the code is an adapter for a named provider quirk
- the value preserves backward compatibility during migration
- the list is a non-authoritative fallback with tests showing the primary mechanism works without it

When hardcoding is acceptable, keep it centralized, documented, and covered by tests.

## Test Requirements

Every non-trivial fix must include at least one test that cannot pass by hardcoding only the observed example.

Add tests from at least two categories when the issue is open-world:

- same intent, different language or phrasing
- same behavior, different provider/backend output shape
- positive case plus negative case where no obligation should be inferred
- final-answer guard where no successful mutation/action occurred
- malformed or partial structured output
- regression test for the exact reported case

For artifact or mutation requests, verify the observable outcome, not just the route label. Example: assert that a file write event succeeded or the file exists with expected content, not only that intent was classified as write.

## Review Checklist

Reject or revise a fix when any item is true:

- It only adds the user's exact phrase, filename, selector, provider string, or screenshot text.
- It adds a keyword list as the primary decision mechanism for open-ended user intent.
- It passes because the test repeats the same literal that was added to code.
- It claims success without verifying the requested side effect.
- It spreads special cases across unrelated modules instead of adding one adapter or contract.
- It hides uncertainty with fallback success, default zeros, or silent no-ops.
- It adds a semantic layer that runs on every turn, or a new entity, status, or table, without removing or merging an existing one.

Approve when:

- The fix states the invariant being protected.
- The implementation lives at the right abstraction boundary.
- The tests include a counterexample or variant that defeats one-off hardcoding.
- Any remaining hardcoded values are justified as closed-set or adapter-local.

## Response Pattern

When reporting work on a relevant fix, include a concise note:

- Classification: closed set, open set, or adapter boundary
- Chosen abstraction: parser, obligation, verifier, capability model, registry, or policy object
- Anti-hardcode test: the variant or negative case added
- Remaining hardcodes: why they are acceptable, or none

