# 013 — Composites flatten into dotted handles

## Goal

A referenced workflow becomes nodes in the caller's own graph, so a composite adds work without adding
a scope an edge would have to cross.

## Example

Today only an all-leaf definition materializes. When this story closes, a reference expands at resolve
time.

```text
$ ripwork resolve --run-dir /runs/50 --key release --task-file /runs/50/task.md --json
{"schema":"ripwork.resolve.v1","ok":true,"run_dir":"/runs/50","state":"running",
 "nodes":[{"as":"build","needs":[]},
          {"as":"checks.lint","needs":["build"]},
          {"as":"checks.test","needs":["build"]},
          {"as":"publish","needs":["checks.lint","checks.test"]}]}

$ ls /runs/50
build  checks.lint  checks.test  publish  state.json  task.md
```

## Core

Flattening is static and complete at resolve. Once a run is materialized, the set of nodes that will
run is known, and nothing about a composite is deferred to something that happens later.

## In scope

- Expand a `workflow:` reference into the caller's graph, prefixing each child handle with the call's
  handle.
- Enforce the workspace depth ceiling on the reference chain.
- Keep a composite's own edges pointing at the composite, and resolve them to its children.
- Produce no directory for the composite itself, which runs no agent.

## Out of scope

- Loops, which resolve to one node carrying an unexpanded template and are 014's.
- Any edge that crosses a scope boundary. Edges point at siblings, and a composite is a sibling.
- A finer exclusion mechanism than the edge, which is Q-003's.

## Governed by

- `docs/reference/workflow-definition.md#call-forms` — the three kinds and what a composite carries.
- `docs/reference/workflow-definition.md#edges` — why the edge points at siblings and never crosses a
  scope.
- `docs/decisions/ADR-0006-express-exclusion-with-needs.md` — the accepted cost this story makes
  visible.

## Amends

- `docs/reference/workflow-definition.md` — flattening into dotted sibling handles describes what
  resolve does.
- `docs/reference/glossary.md` — the handle entry describes an expansion that happens.

## Acceptance

- A reference expands into prefixed sibling handles in the caller's graph — `composite_flattens`.
- An edge naming the composite resolves to every child of it — `edge_to_a_composite_resolves`.
- A reference chain deeper than the ceiling is a finding at validate — `depth_ceiling_holds`.
- The composite itself gets no node directory — `composite_writes_nothing`.

## Tasks

- [ ] Answer Q-003, or record that the accepted cost stands.
- [ ] Implement expansion with handle prefixing.
- [ ] Enforce the depth ceiling in the validator.
- [ ] Resolve edges naming a composite to its children.

## Rabbit holes

- Scoped edges look like the natural fix for coarse exclusion — escape: edges point at siblings, and
  changing that reopens a decision rather than implementing one.
- Handle prefixing can collide with an author's own dotted handle — escape: the safe-segment rule
  admits dots, so collision is a validator finding, not a runtime surprise.

## Done when

The named tests pass unskipped, Q-003 has exited, and a two-level reference materializes the graph the
specification describes.

## Revisions

None
