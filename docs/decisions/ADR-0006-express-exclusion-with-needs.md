# ADR-0006: Express step exclusion with needs rather than a marker

## Context and Problem Statement

Two steps may touch the same state and must not overlap. Marking such a step with a dedicated
exclusion key raises a question the key itself cannot answer: what scope the exclusion covers. A
boolean serializes the whole run, and a named lock needs a lock table in run state whose entries a
crashed owner holds forever under a no-timeout rule.

## Considered Options

- A definition-only boolean with whole-run exclusion
- A named lock with a lock table in run state
- No marker; express exclusion with `needs:`

## Decision Outcome

Chosen option: `No marker; express exclusion with needs:`.

`needs:` is the only edge key and carries both order and exclusion. Two steps that must not overlap
get an edge between them. Readiness already withholds a node until its edges are satisfied, so the
capacity exists at zero cost: no new key, no new run state, no lock table, and no exclusion scope to
compute.

One sentence of the grammar carries the cost this creates. An edge with no corresponding data
reference is an ordering constraint, and deleting it changes behaviour while validation still passes.

## Consequences

- Good: every conflict an author can see is expressible today, with nothing new built.
- Good: no state that a crash can leave held.
- Bad: exclusion across a scope boundary is coarse. Two conflicting leaves in different composites can
  only be separated by serializing the composites that contain them.
- Bad: a composite rewritten to touch shared state can break a caller who cannot see its interior.
  Consistency is the author's responsibility, and a harder check waits for a real case rather than
  being built for a hypothetical one.
- Bad: a mutex edge is textually identical to a data edge and is silently deletable.

## Status

Accepted
