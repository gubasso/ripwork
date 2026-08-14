# ADR-0008: Select a step's execution context at its node

## Context and Problem Statement

Some steps must run in a context that has not seen the run — a reviewer that must not see the verdict
it is reviewing — and some may run inside the driver's own context, which is cheaper and keeps the
work in one place. Two of the four capabilities a driver declares are capabilities about exactly this
choice, and a capability needs something to range over. Fixed for a whole workspace, the choice gives
them nothing.

## Considered Options

- Keep the choice a workspace default and scope the capabilities to the whole run
- Promote the choice to an optional key on a node
- Derive the choice from the engine's provider

## Decision Outcome

Chosen option: `Promote the choice to an optional key on a node`.

A `step:` MAY carry `context:` taking the literal `fresh` or `inherit`. Absent, the node takes the
workspace default. The value is a literal, never an expression, and there is no call-site override
map — the same one-writer discipline [ADR-0004](./ADR-0004-select-an-engine-at-the-definition-or-its-call-site.md)
chose for `engine:`. A `workflow:` or `loop:` node carries no context key, because it runs no agent;
its steps each carry their own.

Deriving it from the provider was rejected because a same-provider fork is a legitimate reason to
cross a boundary. A reviewer forked from a coordinator on the same provider crosses precisely so that
it never sees the coordinator's verdict. Context is a boundary question, not a vendor one.

## Consequences

- Good: one definition may mix an in-session judgment with a forked worker, which is the shape a real
  workflow has.
- Good: a driver that can only fork, and a driver that can only run in session, each get a run they
  can honour rather than a silent substitution.
- Bad: an author now has a third per-step key to get right, and getting it wrong fails a node closed
  rather than degrading.

## Status

Accepted
