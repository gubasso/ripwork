# ADR-0003: A workflow is a directed acyclic graph of three call forms

## Context and Problem Statement

The grammar has to express a real multi-step agent workflow — a plan reviewed and then applied, a
review cycle that repeats until it settles — without becoming a scripting language. A format admitting
conditionals, matrices, and expressions acquires a runtime, and each of those features is a place
where a definition can be correct and still fail after real spend.

## Considered Options

- A general graph with conditionals and an expression language
- A linear sequence only, with repetition pushed inside a step
- Three call forms, one edge key, and no conditional

## Decision Outcome

Chosen option: `Three call forms, one edge key, and no conditional`.

A workflow is a list of entries. Each carries exactly one kind key: `step:` for a leaf that runs one
agent, `workflow:` for a composite by reference, and `loop:` for a composite inline. `needs:` is the
only edge key, it points at siblings, and it never crosses a scope boundary.

A unit whose repetition count is unknown until something runs is an ordinary leaf whose brief owns its
internal loop. That is the one shape the grammar admits for it, and it needs no new key.

Recorded so it is not re-proposed: no conditional of any kind, so every node in the run graph runs and
the join question never arises; no `matrix:`; no fold operator; no `for_each:`; and no expression
language anywhere.

## Consequences

- Good: validation runs once at load, before any agent is spawned, so a structural failure never lands
  at depth after real spend.
- Good: with no conditional, the set of nodes that will run is known once the run is materialized.
- Bad: a workflow cannot branch on a result. Branching work lives inside one step, where the brief
  owns it and the graph cannot see it.
- Bad: such a step reaches the driver with no per-round decision point, and its round ceiling is prose
  rather than a validated one.

## Status

Accepted
