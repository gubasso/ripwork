# 018 — A driver declares its capabilities

## Goal

A driver says once what it can do, and never gets handed work it cannot honour.

## Example

Today a declaration is stored and read by nothing. When this story closes, it is enforced.

```text
$ ripwork resolve --run-dir /runs/70 --key mixed --task-file /runs/70/task.md \
    --orchestrator '{"name":"sh","version":"1","capabilities":["subprocess"]}' --json
{"ok":true,"state":"running"}

# a node declaring context: inherit, from a driver that cannot run one
$ ripwork next --run-dir /runs/70 --json
{"error":"capability not declared",
 "detail":"node judge declares context: inherit and the driver did not declare inline"}
$ echo $?
2
```

## Core

ripwork enforces rather than trusts. A capability nobody declared is a capability nobody exercises, so
an under-declaring driver gets a run it can honour and an over-declaring one fails at the first
directive it cannot meet, rather than degrading somewhere nobody is watching.

## In scope

- Accept the declaration at resolve, refusing an unknown capability as invalid input.
- Enforce all four absent-behaviours: at most one dispatch directive without `parallel`, an `inherit`
  node failing closed without `inline`, only `inherit` nodes dispatchable without `subprocess`, and a
  judgment directive becoming a hard failure without `ask-user`.
- Emit the judgment directive where the grammar produces one, which is a loop's criterion reaching a
  round boundary.
- Refuse a dispatch that would exceed a declared depth ceiling, and enforce none when none is
  declared.

## Out of scope

- Inventing a capability. The vocabulary is closed at four.
- Supplying a default depth ceiling. Absent, the driver owns whatever limit its own runtime has.
- Guessing what a driver meant. An under-declaration is honoured, not corrected.

## Governed by

- `docs/reference/orchestrator-contract.md#the-capability-vocabulary` — the four members and each
  one's absent-behaviour.
- `docs/reference/orchestrator-contract.md#depth` — why the ceiling is opt-in and never a supplied
  number.
- `docs/decisions/ADR-0008-select-an-execution-context-at-its-node.md` — why two capabilities range
  over a per-node key.

## Amends

- `docs/reference/orchestrator-contract.md` — capability enforcement and the judgment directive stop
  being marked as specified ahead of the runtime.

## Acceptance

- An unknown capability at resolve is invalid input — `capability_vocabulary_is_closed`.
- Each of the four absent-behaviours holds independently — `absent_capability_fails_closed`.
- A driver declaring both context capabilities is handed a definition mixing them — `mixed_context_is_dispatchable`.
- A declared depth ceiling refuses the dispatch that would exceed it — `declared_depth_is_enforced`.
- No ceiling is enforced when none is declared — `absent_depth_enforces_nothing`.

## Tasks

- [ ] Validate the declaration at resolve.
- [ ] Enforce each absent-behaviour at the verb that would exercise it.
- [ ] Emit the judgment directive at a round boundary.
- [ ] Enforce the declared depth ceiling.

## Rabbit holes

- An under-declaration looks like something to helpfully correct — escape: declaration is not a hint,
  and second-guessing it is the failure mode the rule exists to prevent.
- A default depth ceiling looks safer than none — escape: a supplied number is a claim about a runtime
  ripwork knows nothing about.

## Done when

The named tests pass unskipped and each capability is proven to change behaviour by its absence alone.

## Revisions

None
