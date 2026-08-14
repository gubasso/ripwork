# 007 — resolve materializes the run

## Goal

One call turns a validated definition into a run directory that a driver can work against, and an
invalid definition leaves nothing behind.

## Example

Today validation is the end of the road. When this story closes, a run exists on disk.

```text
$ ripwork resolve --run-dir /runs/42 --key plan-review-apply \
                  --task-file /runs/42/task.md --json
{"schema":"ripwork.resolve.v1","ok":true,"action":"resolve","run_dir":"/runs/42",
 "key":"plan-review-apply","state":"running",
 "nodes":[{"as":"draft","id":"plan","dir":"/runs/42/draft","needs":[],"engine":"<engine>"},
          {"as":"vet","id":"review","dir":"/runs/42/vet","needs":["draft"],"engine":"<engine>"},
          {"as":"apply","id":"apply","dir":"/runs/42/apply","needs":["draft","vet"],
           "engine":"<engine>"}]}

$ ls /runs/42
apply  draft  state.json  task.md  vet
```

## Core

Validate first, materialize second. A definition with any finding creates no directory and writes no
state, so a failed resolve leaves a directory a caller can retry into rather than a half-built run
they have to recognize and clean up.

## In scope

- Validate the definition and refuse before creating anything.
- Write the run-state document atomically, with the nodes, the workspace defaults, the task file, and
  the definition digest.
- Create one directory per node, named by its handle.
- Store the driver declaration and the depth ceiling as given, without reading either.
- Report every path as absolute.

## Out of scope

- Reading the declaration or the ceiling. Enforcement is 018's, and storing them now is what lets 018
  be written against a fixed target.
- Composites and loops. A linear all-leaf definition is what this story materializes.
- Dispatching anything.

## Governed by

- `docs/reference/command-surface.md#resolve` — the arguments and the document.
- `docs/reference/run-directory.md` — the layout, the run-state document, and the digest.
- `docs/decisions/ADR-0009-keep-one-writer-for-run-state.md` — why the write is atomic and why a
  partial state document is worse than none.

## Amends

- `docs/reference/run-directory.md` — the run-state document and the layout describe files that exist.
- `docs/reference/command-surface.md` — the resolve section describes a verb that exists.

## Acceptance

- A definition with any finding creates no directory and reports `2` — `invalid_leaves_nothing`.
- A valid definition writes state and one directory per node — `valid_materializes_every_node`.
- Every path in the document is absolute, and the state's node `dir` stays relative to the run
  directory as `docs/reference/run-directory.md` specifies — `paths_are_absolute`.
- A handle that is not a safe path segment cannot name a directory outside the run directory —
  `no_directory_escapes_the_run`.
- The declaration and the ceiling are stored as given and read by nothing — `declaration_is_stored_verbatim`.

## Tasks

- [ ] Order validation before materialization and prove the ordering with a failing definition.
- [ ] Write the run-state document atomically.
- [ ] Create node directories behind a second safe-segment check.
- [ ] Emit the resolve document.

## Rabbit holes

- Partial materialization can look recoverable — escape: nothing is created until validation has
  passed entirely.
- A safe-segment check can be assumed done by the validator — escape: check again at the point a
  directory is created, because that is where the damage would land.

## Done when

The named tests pass unskipped, and a failed resolve is proven to leave an empty directory.

## Revisions

None
