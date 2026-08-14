# ADR-0011: Make the exit protocol protocol only

## Context and Problem Statement

A driver reads two things from every call: an exit code and a document. Encoding a run's terminal
state into the exit code makes the two overlap, and the overlap is not harmless — a run that ended
`failed` is a run ripwork reported correctly, so the call succeeded and the run did not. Collapsing
those into one number forces every driver to guess which sense a given code carries.

## Considered Options

- Encode terminal states in the exit code, one code per state
- Use only success and failure, and put everything else in the document
- Four codes describing the call, with terminal state in the document

## Decision Outcome

Chosen option: `Four codes describing the call, with terminal state in the document`.

- `0` — a valid result, or an applied transition. Including a correctly reported terminal `failed`.
- `1` — an internal failure reading or persisting state.
- `2` — invalid input: a stale or missing token, an illegal transition, a digest mismatch, or
  malformed arguments.
- `75` — cannot advance yet. Nothing is wrong; the caller re-issues.

The code describes the call. The run's terminal state lives in the document's state field and nowhere
else. `75` is the retry signal a driver loops on, and it is the same signal a long-running job's
poll uses, so one retry shape covers both.

## Consequences

- Good: a driver's retry loop is one comparison, and reporting a terminal state cannot be confused
  with failing to report it.
- Good: an invalid input and an internal fault are distinguishable, so a driver knows whether
  re-issuing could ever help.
- Bad: a driver that branches on the exit code alone reads a terminal `failed` run as a healthy one.
  The contract has to say so explicitly, because the failure is silent.
- Bad: four codes is a vocabulary an implementation must not extend. A fifth code is a breaking
  change to every driver.

## Status

Accepted
