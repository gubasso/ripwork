# 012 — A composite and a loop survive a restart

## Goal

A run containing a referenced workflow and a judged loop reaches a terminal state after its driver is
killed mid-flight, without dispatching any node twice.

## Example

Today a loop resolves to nothing and a killed driver leaves a run nobody can read. When this end state
is reached, the driver comes back and the record tells it exactly what happened.

```text
# round 2 is claimed; the driver is killed
$ ripwork next --run-dir /runs/51 --json
{"schema":"ripwork.next.v1","ok":false,"state":"running","node":null,
 "reason":"review-cycle.2.fix is claimed by sh-1 and has no receipt"}
$ echo $?
75

$ ripwork reclaim --run-dir /runs/51 --as review-cycle.2.fix \
                  --previous-claim <old> --owner sh-2 --reason "driver restarted" --json
{"schema":"ripwork.reclaim.v1","ok":true,"claim_token":"<new>"}

# the round completes; the driver reads until: and the round's directories, then judges
$ ripwork advance --run-dir /runs/51 --loop review-cycle --decision-token <tok> \
                  --outcome converged --reason criterion-met --json
{"schema":"ripwork.advance.v1","ok":true,"round":2,"outcome":"converged","state":"running"}
```

## Core

No node is dispatched twice, under any interleaving of crash and restart. A claimed node with no
receipt is reported as unterminated rather than as lost or as done, and the only way to take it is a
reclaim the record keeps.

## Out of scope

- Retries, backoff, and distributed locking. Recovery is a driver reading the record, not a scheduler.
- A second loop form, or any expression over a round's contents.
- Time-based ownership. Nothing expires, and duration is never a failure signal.

## Governed by

- `docs/reference/workflow-definition.md#loops` — the loop rules and what a round boundary requires.
- `docs/reference/run-directory.md#crash-semantics` — what an absent receipt means and what a driver
  does about it.
- `docs/decisions/ADR-0007-judge-loop-convergence-with-a-prose-criterion.md` — who judges convergence
  and what ripwork still enforces without reading an artifact.
- `docs/decisions/ADR-0009-keep-one-writer-for-run-state.md` — why recovery is reconstruction from
  files rather than supervision.

## Amends

- `docs/reference/run-directory.md` — the four restart fields and the crash semantics stop being
  specified ahead of the runtime.
- `docs/reference/workflow-definition.md` — the loop rules describe a materializer that exists.

## Done when

A definition containing one referenced workflow and one loop reaches a terminal state after a kill in
every phase of the lifecycle, with one dispatch per node across every run, and every round's decision
readable afterwards with its reason.

## Revisions

None
