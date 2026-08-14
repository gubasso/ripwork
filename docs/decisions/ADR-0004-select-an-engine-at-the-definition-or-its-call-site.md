# ADR-0004: Select a step's engine at its definition or its direct call site

## Context and Problem Statement

Every leaf step needs one execution target: a provider, a model, and an effort. The question is how
many writers may set it. An arrangement in which a definition, a caller reaching into a referenced
workflow, and a global flag can all set the same value needs a precedence rule, and a precedence rule
is a thing every reader must hold in their head to know what a definition will actually run.

## Considered Options

- A global flag above everything, overriding per run
- A caller-supplied map that reaches into a referenced workflow's interior
- A required key on the definition and one optional override at the direct call site

## Decision Outcome

Chosen option: `A required key on the definition and one optional override at the direct call site`.

A step definition declares a required `engine:` naming one record in the registry. Its direct call
site MAY declare an optional `engine:`, which wins for that call only. Nothing else sets it. Values
are literals in both positions, never expressions.

A `workflow:` or `loop:` node carries no engine key, because it runs no agent. A call site overrides
only the step it calls and never reaches into a referenced workflow's interior, so the whole
precedence rule is one sentence: the direct call site wins, and nothing else sets the value.

## Consequences

- Good: reading a step definition and its call site tells a reader what will run. There is no third
  place to check.
- Good: one override level is enough for fan-out, which is the case that motivated the distinction.
- Bad: varying the engines inside a referenced workflow means authoring a variant file rather than
  passing a map.
- Bad: a run cannot be retuned wholesale from the command line. Changing every engine is an edit.

## Status

Accepted
