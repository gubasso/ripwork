# 017 — An orchestrator proves its conformance

## Goal

A candidate driver is conformant because it drove a fixture to a known result, not because it says it
is.

## Example

Today the contract is prose and nothing tests a claim against it. When this end state is reached, the
weakest possible driver is the one that proves the contract.

```text
$ ripwork conformance --run-dir /runs/fixture
driver        sh-fixture 1
capabilities  subprocess
ordering      ok      3 nodes, 3 dispatches
loop          ok      exhausted at max_rounds 2, reported failure
pause         ok      resumed to terminal
stale token   ok      refused, exit 2
crash         ok      1 reclaim, no double dispatch
result        conformant
```

## Core

The fixture makes no vendor assumption. It runs a shell script that dispatches with an ordinary
subprocess, because that is the floor the contract promises — a fixture needing anything more would be
testing a runtime rather than the contract.

## Out of scope

- Provider authentication, model policy, and any real agent. The fixture dispatches something trivial
  and deterministic.
- Performance measurement.
- Judging a real convergence. The fixture scripts its driver's outcomes, because a real judgment is
  not deterministic and a known result cannot assert one.

## Governed by

- `docs/reference/orchestrator-contract.md` — the declaration, the capability vocabulary, the
  obligations, and the prohibitions the fixture exercises.
- `docs/decisions/ADR-0002-ripwork-coordinates-and-never-dispatches.md` — why the floor is a shell
  script and why raising it is a regression.
- `docs/decisions/ADR-0011-make-the-exit-protocol-protocol-only.md` — what each exit code asserts, and
  why a terminal state is read from the document.

## Amends

- `docs/reference/orchestrator-contract.md` — capability enforcement and the `requires_judgment`
  directive stop being marked as specified ahead of the runtime.

## Done when

A scripted shell driver reaches the known result across ordering, loop exhaustion, pause and resume,
a stale token, and crash recovery, asserting state transitions and not only final output.

## Revisions

None
