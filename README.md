# Anti-Hardcode Engineering

A Codex skill for preventing brittle, one-off engineering fixes before they enter the codebase.

This skill is intended for implementation-time use, not only after-the-fact review. It helps an agent decide whether a change is a closed-set mapping, an open-world/generalizable behavior, or an adapter-boundary compatibility case before writing code.

## What It Prevents

- Patching natural-language intent bugs with language-specific keyword lists as the source of truth
- Fixing parser or tool-call issues with scattered substring checks
- Solving UI, scraping, or extraction bugs with brittle selectors or copied examples
- Adding provider-, path-, or screenshot-specific conditionals without an abstraction boundary
- Writing tests that only prove the observed bug report was hardcoded correctly

## Core Principle

Use hardcoded values only for closed sets, such as official enums, protocol statuses, API fields, centralized configuration, or adapter-local provider quirks.

For open-world problems, prefer durable mechanisms such as:

- obligations
- capabilities
- contracts
- parsers
- verifiers
- registries
- policy objects
- adapter boundaries

## Typical Use Cases

Use this skill when implementing or reviewing:

- bug fixes
- agent/tool runtime behavior
- intent detection and routing
- parser behavior
- validation gates
- policy checks
- UI extraction
- scraping or document extraction
- any change that could be solved too narrowly by hardcoding the current example

## When Not To Use

This skill targets a specific open-world bug at one boundary; it is not an architecture. Don't install its obligation/verifier pipeline as a global per-turn or per-subagent layer, don't predict obligations and then gate them when exposed tools can define capability, and check reference implementations or existing primitives before adding new contracts, entities, or states. See the "When NOT To Use" section in `SKILL.md`.

## Installation

Copy this folder into your Codex skills directory:

```powershell
C:\Users\Xu\.codex\skills\anti-hardcode-engineering
```

Or install it into any configured skills path as:

```text
anti-hardcode-engineering/
  SKILL.md
  agents/openai.yaml
```

## Usage

Reference the skill explicitly in a coding task:

```text
Use anti-hardcode-engineering while implementing this fix.
```

The skill asks the agent to report:

- classification: closed set, open world, or adapter boundary
- chosen abstraction: parser, obligation, verifier, registry, policy object, etc.
- anti-hardcode test: a variant or negative case that defeats one-off hardcoding
- remaining hardcodes: why they are acceptable, or none

## Files

- `SKILL.md` - the actual skill instructions
- `agents/openai.yaml` - UI metadata for the skill
