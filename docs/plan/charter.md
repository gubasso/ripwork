# ripwork — Charter

## What this is for

ripwork exists so a multi-step agent workflow can be run by a program that knows nothing about
agents. It owns the deterministic half — validation, run materialization, readiness, claims,
receipts, and round boundaries — and leaves dispatch and judgment to whatever drives it.

The specification is written. What this record plans is an implementation of it, built in the order
that makes each layer provable before the next one rests on it.

## Pillars

- Decidable without reading an artifact. Everything ripwork enforces is true or false without opening
  a file a producer wrote.
- One writer per durable artifact, so a run is reconstructable after its driver dies.
- Vendor-neutral to the point of inconvenience. A shell script is a conformant driver, and any
  obligation it could not meet is a regression.
- Fail before spend. A structural error is reported at load, not at depth after a run has cost
  something.

## No-gos

- Evaluating a stopping criterion, or any expression, anywhere.
- Enforcing at run time what a probabilistic producer was asked to write.
- Scheduling, retrying, backing off, or acting without a driver's call.
- Storing, rotating, or selecting a credential.
- Naming a vendor, model, or program in the specification.

## Iteration

Two weeks. One iteration is a bounded round of implementation and the verification that closes it.
`.plan-xp.yml` carries the same cadence in the form a tool reads. Changing it restarts the velocity
series, so it stays fixed unless the shape of the work changes.
