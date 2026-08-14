# 008 — next reports the frontier

## Goal

A driver asks what to run and is told, or is told why nothing is runnable yet, without ever deriving a
path or a readiness rule itself.

## Example

Today a run exists and nothing says what to do with it. When this story closes, one call answers.

```text
$ ripwork next --run-dir /runs/42 --json
{"schema":"ripwork.next.v1","ok":true,"action":"next","state":"running",
 "node":{"as":"vet","id":"review","dir":"/runs/42/vet","engine":"<engine>",
         "brief":"review","inputs":["/runs/42/draft/"]}}

# nothing ready, run still live
$ ripwork next --run-dir /runs/42 --json
{"schema":"ripwork.next.v1","ok":false,"state":"running","node":null,
 "reason":"a claimed node blocks the frontier"}
$ echo $?
75
```

## Core

`inputs` is ripwork's answer, not the driver's. A driver hands a node exactly the upstream directories
it was given, so the mapping from edges to inputs has one owner and cannot drift between drivers.

## In scope

- Report one ready node: pending, with every edge target at `done`.
- Name the upstream directories for that node as absolute paths.
- Report `75` with a reason when the run is live and nothing is ready.
- Report the run's state, recomputed from the node statuses, when the run is not live.

## Out of scope

- Returning a ready set. Absent the `parallel` capability at most one node is returned, and that
  capability is 018's.
- Any scheduling policy beyond readiness. Which ready node comes first is not a promise.
- Claiming. Reporting a node does not reserve it.

## Governed by

- `docs/reference/command-surface.md#next` — the document, the readiness rule, and the `75` case.
- `docs/reference/run-directory.md` — the node record the frontier reads.
- `docs/decisions/ADR-0011-make-the-exit-protocol-protocol-only.md` — why waiting is `75` and not an
  error.

## Amends

- `docs/reference/command-surface.md` — the next section describes a verb that exists.

## Acceptance

- A node whose every edge target is `done` is reported — `ready_means_edges_are_done`.
- A live run with nothing ready reports `75` and a reason — `blocked_frontier_is_temporary`.
- A terminal run reports its state and a null node at `0` — `terminal_reports_state`.
- The reported inputs are exactly the upstream node directories, absolute — `inputs_are_named_not_derived`.
- The run state is recomputed in one place, so a failed upstream cannot be reported as running —
  `state_has_one_owner`.

## Tasks

- [ ] Implement readiness over the node records.
- [ ] Implement state recomputation once and call it from everywhere.
- [ ] Emit the frontier document and the `75` case.

## Rabbit holes

- A general scheduler can grow out of readiness — escape: readiness is the whole rule, and ordering
  among ready nodes is deliberately unpromised.
- A second copy of the state rule drifts silently — escape: one owner, called from every verb that
  reports a state.

## Done when

The named tests pass unskipped and a failed upstream is proven to make the run terminal in every verb
that reports a state.

## Revisions

None
