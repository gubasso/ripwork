# 019 — The conformance fixture reaches a golden result

## Goal

A candidate driver is conformant because it drove a fixture to a known result, not because it claims
to be.

## Example

Today the contract is prose and a claim against it is unfalsifiable. When this story closes, the claim
is a run.

```text
$ ripwork conformance --run-dir /runs/fixture
driver        sh-fixture 1
capabilities  subprocess
ordering      ok      3 nodes, 3 dispatches, edges respected
loop          ok      exhausted at max_rounds 2, terminal failed
pause         ok      paused at round 1, resumed to terminal
stale token   ok      refused, exit 2
crash         ok      1 reclaim, no double dispatch
result        conformant
```

## Core

The fixture's driver is a shell script dispatching with an ordinary subprocess. That is the floor the
contract promises, and a fixture that needed anything more would be testing a runtime rather than the
contract.

## In scope

- A scripted driver covering ordering, loop exhaustion, pause and resume, a stale token, and crash
  recovery.
- Asserting state transitions as well as final output, so a golden result cannot pass while the run
  took a different path to it.
- Scripting the driver's loop outcomes, because a real convergence judgment is not deterministic and a
  known result cannot assert one.
- The conformance readout, which is the one verb with no machine-readable document.

## Out of scope

- Any real agent, provider, credential, or model.
- Performance measurement.
- Asserting that a driver made a good judgment. The fixture asserts that it made one and reported it
  legally.

## Governed by

- `docs/reference/orchestrator-contract.md` — the obligations and prohibitions the fixture exercises.
- `docs/reference/command-surface.md#conformance` — why the readout has no machine-readable document.
- `docs/decisions/ADR-0002-ripwork-coordinates-and-never-dispatches.md` — why the floor is a shell
  script and raising it is a regression.

## Amends

- `docs/reference/orchestrator-contract.md` — the contract names the fixture a candidate driver
  passes.
- `docs/reference/command-surface.md` — the conformance section describes a verb that exists.

## Acceptance

- A shell driver with only the subprocess capability reaches the known result — `shell_driver_is_conformant`.
- Every branch is asserted on its transitions, not only its final output — `transitions_are_asserted`.
- A driver that edits state directly fails the fixture — `state_editor_is_not_conformant`.
- A driver that treats a terminal `failed` as success fails the fixture — `only_done_is_success`.

## Tasks

- [ ] Write the scripted driver.
- [ ] Cover each of the five branches.
- [ ] Assert transitions per branch.
- [ ] Emit the conformance readout.

## Rabbit holes

- Vendor details can leak into a fixture through convenience — escape: the driver is a shell script,
  and anything it cannot do is out.
- A golden result can hide a semantic omission — escape: assert the transitions, not only the bytes.

## Done when

The named tests pass unskipped and each of the two negative cases is proven to fail the fixture.

## Revisions

None
