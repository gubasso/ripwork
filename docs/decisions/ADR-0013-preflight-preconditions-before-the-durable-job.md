# ADR-0013: Preflight provider preconditions before the durable job

## Context and Problem Statement

A runner has to distinguish a launch that never happened from a run that happened and failed. Some
provider launchers replace themselves with the child process rather than supervising it, so they
leave no post-flight step, and their own exit codes overlap the range the agent itself can report.
Once a durable job exists, its status belongs to the agent, and a missing credential becomes
indistinguishable from a refusal the agent produced.

## Considered Options

- Classify after the fact from the captured streams and exit code
- Parse the provider's own error prose to recognize a failed launch
- Check the standing preconditions before the durable state document exists

## Decision Outcome

Chosen option: `Check the standing preconditions before the durable state document exists`.

A runner reports version and authentication preconditions while it is still the thing answering:
before it creates the durable state document, and therefore before there is a job whose status could
be confused with the check. Reportable preconditions are the standing ones a retry reproduces — a
missing program, an unbound or unselected account, an unresolved profile, a version below a floor the
provider enforces, and an absent or refused credential.

Each is reported with what was checked and what would satisfy it. A runner fixes none of them.
Creating, storing, rotating, and selecting credentials stay out of scope.

Parsing provider error prose was rejected outright: it makes a runner's correctness depend on strings
a provider may reword in any release.

## Consequences

- Good: never launched and ran and failed stay distinguishable without reading a provider's
  diagnostics.
- Good: a caller learns a launch is impossible before any spend, rather than after.
- Bad: a precondition that only appears mid-run is still invisible to the check, so preflight narrows
  the window rather than closing it.
- Bad: every runner carries preflight work proportional to its provider's binding model, and a
  provider with no such model still pays for the structure.

## Status

Accepted
