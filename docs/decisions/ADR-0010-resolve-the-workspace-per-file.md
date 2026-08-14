# ADR-0010: Resolve the workspace per file, and keep the engine registry out of it

## Context and Problem Statement

A user needs to author a workflow that reuses steps and briefs someone else shipped, and to replace
one of them without forking the rest. That calls for layers. It also raises a question the layering
scheme cannot dodge: whether a lower layer merges into a higher one, and whether the engine registry
is one of the things a layer may contribute.

## Considered Options

- Resolve per root: the first layer holding a workspace wins, whole
- Resolve per file, merging the keys inside a file that two layers both hold
- Resolve per file, taking each file whole, with the registry outside the layered tree

## Decision Outcome

Chosen option: `Resolve per file, taking each file whole, with the registry outside the layered tree`.

The unit of resolution is the file. One resolver walks the layers in a fixed order and returns the
first file that exists, taken whole. Nothing is merged — not two files, and not the keys inside them
— so a project-layer workflow replaces the shipped one entirely rather than inheriting its steps.
Resolving per file rather than per root is what lets a user-authored workflow reference a shipped
step, and `show` reports the resolved source per file so shadowing stays visible rather than
inferred.

The engine registry deliberately lives outside that tree. Membership in the registry is the
permission to dispatch, so a layer that could add its own rows could grant itself an engine.

## Consequences

- Good: overriding one step costs one file, and the override is visible in a command's output rather
  than inferred from layer order.
- Good: no merge semantics to specify, and no partially-inherited definition to debug.
- Bad: overriding a workflow means restating its steps, so a shipped workflow gaining a step does not
  reach a project that shadowed it.
- Bad: a project cannot add an engine of its own. Widening the registry is a change to whatever ships
  it.

## Status

Accepted
