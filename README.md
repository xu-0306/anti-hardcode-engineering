# Engineering Guardrails

Skills for coding agents (Codex, Claude Code, and other agents that load `SKILL.md` folders) that keep changes from going wrong in two opposite directions.

| Skill | Prevents | Use when |
|---|---|---|
| [anti-hardcode-engineering](anti-hardcode-engineering/) | fixes that are too narrow: keyword lists, copied examples, brittle selectors, provider strings | fixing a specific open-world bug at one boundary |
| [anti-complexity-engineering](anti-complexity-engineering/) | designs that are too big: extra entities, parallel lifecycles, predict-then-gate layers, plans that only add | designing features, writing plans, reviewing changes that add concepts |

They are meant to be used together. anti-hardcode-engineering chooses the right abstraction for one bug; anti-complexity-engineering keeps that abstraction from spreading into a global layer, and its concept budget takes precedence when the two conflict.

## anti-complexity-engineering in short

1. Reference alignment: check how a mature implementation does the same feature before designing.
2. Primitive mapping: express the feature with existing primitives first.
3. Concept budget: every new entity, status, table, field, or layer needs an observed requirement.
4. Deletion ledger: every change says what it deletes or merges.
5. Proportional planning: plan size matches the change.

Security, data-loss prevention, real concurrency issues, required audit, and measured performance work are explicitly kept.

## Installation

Copy one or both skill folders into your skills directory, for example:

```text
~/.codex/skills/anti-hardcode-engineering/
~/.codex/skills/anti-complexity-engineering/
~/.claude/skills/anti-complexity-engineering/
```

Each folder contains `SKILL.md` and `agents/openai.yaml` (UI metadata).

## Usage

```text
Use anti-complexity-engineering while designing this feature.
Use anti-hardcode-engineering and anti-complexity-engineering while fixing this bug.
```
