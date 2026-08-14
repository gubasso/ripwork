# ADR-0009: Keep one writer for every run-state artifact

## Context and Problem Statement

A run has to survive its driver crashing or being replaced mid-flight. What makes that possible is not
a supervisor — a supervisor is another process that can also die — but a durable record of what
happened. A record two parties may write can disagree with itself, and after a crash nobody is left
who knows which writer was last.

## Considered Options

- Let a driver write run state directly, with ripwork validating it afterwards
- Let a driver append events that ripwork folds into state
- One writer per artifact, enforced by tokens

## Decision Outcome

Chosen option: `One writer per artifact, enforced by tokens`.

The agent owns its own node directory. ripwork owns the run-state document and every receipt. A
driver reads both and writes neither. Ownership of a node is taken with `claim`, which mints a token;
`record` refuses a token that is not the live one, and `reclaim` is the only way to take a node whose
owner did not return.

Every state write is atomic, because a partial state document is worse than none. Persisting or
reading state is the only thing that reports an internal failure; everything a caller got wrong is
an invalid input.

The run-state document is also the recovery surface. An absent receipt means the node never
terminated, so the frontier reports it as running with its lease age, and the driver reclaims it.

## Consequences

- Good: a run's outcome is reconstructed from files, so no observer needs to survive.
- Good: a double dispatch is structurally impossible without a reclaim, which is itself recorded.
- Bad: a driver that edits state directly is not a slower conformant driver; it is a driver whose run
  cannot be recovered. The prohibition has to be stated, because nothing prevents the edit.
- Bad: ownership never expires. A node whose owner vanished stays claimed until a driver reclaims it,
  and no elapsed time does that on its own.

## Status

Accepted
