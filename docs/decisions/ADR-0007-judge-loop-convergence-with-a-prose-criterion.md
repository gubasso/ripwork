# ADR-0007: Judge loop convergence with a prose criterion

## Context and Problem Statement

A loop needs a stopping criterion. Evaluating one as an expression while the driver separately reports
an outcome leaves two authorities for one verdict. It also needs values to compare, and with declared
artifact handles removed there are none — the criterion's only remaining subject is what the round's
directories contain, which only a reader can assess.

## Considered Options

- Keep an expression dialect for the criterion
- Evaluate a predicate over a reserved file the agent writes
- Carry the criterion as prose the orchestrator judges

## Decision Outcome

Chosen option: `Carry the criterion as prose the orchestrator judges`.

`until:` is a string ripwork stores and never parses. At each round boundary the driver reads the
criterion and the round's directories, then reports through `advance`, which is the only writer of
round N+1 and carries a closed outcome vocabulary beside a closed reason vocabulary.

ripwork still enforces four things without reading one artifact: `max_rounds:` as a hard ceiling
checked before a round is materialized, the decision token, both closed vocabularies, and round
completeness — a round that produced no files refuses to continue or converge.

Reaching the ceiling is a failure, not a success. An authored criterion SHOULD describe the stall path
as well as the success path.

## Consequences

- Good: the expression dialect is deleted with no remaining evaluation site, so the grammar cannot
  drift back into a scripting language.
- Good: the driver's report is the sole verdict, and every one is recorded with its reason.
- Bad: a premature convergence stays possible. It is auditable from the per-round record and is not
  prevented.
- Bad: a conformance fixture must script its driver's outcomes, because a real judgment is not
  deterministic and a golden result cannot assert one.

## Status

Accepted
