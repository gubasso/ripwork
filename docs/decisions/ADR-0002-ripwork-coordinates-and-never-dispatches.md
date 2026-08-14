# ADR-0002: ripwork coordinates a run and never dispatches one

## Context and Problem Statement

A workflow of agent steps needs something to decide what runs next, something to run it, and something
to judge the result. A tool taking all three becomes a client of every agent runtime it supports. A
tool taking none is a schema. The question is which of the three ripwork keeps.

## Considered Options

- Own dispatch too: ripwork launches the agent for each step
- Own judgment too: ripwork reads artifacts and decides whether a step succeeded
- Own only what is decidable without reading an artifact or launching a process

## Decision Outcome

Chosen option: `Own only what is decidable without reading an artifact or launching a process`.

ripwork validates a definition, materializes a run directory, reports which node is ready, issues and
verifies claims, inventories what a node directory holds, and gates every round boundary. It counts
files and never reads them. It launches nothing and judges nothing.

A driver — any program that reaches a terminal state through the verbs — owns dispatch and judgment.
The [orchestrator contract](../reference/orchestrator-contract.md) is the whole of what one owes, and
it is deliberately satisfiable by a shell script. Anything added to those obligations that a shell
script cannot meet is a contract regression.

The line sits here because a probabilistic producer cannot be held to a contract about its output.
Everything ripwork enforces is a fact about structure, ordering, tokens, counts, and ceilings, each
true or false without opening a file.

## Consequences

- Good: no vendor, model, account, or transport appears anywhere in the specification.
- Good: a run is reconstructable from files after a crash, because the durable record never depended
  on an observer that may not have survived.
- Bad: a driver is mandatory. ripwork alone runs nothing, and a user with no driver has a validator.
- Bad: a driver can report a premature judgment, and ripwork will record it. The record is auditable
  after the fact; it is not prevented.

## Status

Accepted
