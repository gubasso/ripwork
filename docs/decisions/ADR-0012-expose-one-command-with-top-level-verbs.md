# ADR-0012: Expose one command whose verbs are top-level

## Context and Problem Statement

The design arrived from a host tool where it was one subcommand among many, so every verb was spelled
under a namespace. ripwork is not a host tool with a workflow feature; running a workflow is the
whole of what it does. The namespace has to be either justified on its own terms or dropped, and the
same question decides whether the vocabulary keeps a vendor-shaped word for the prose an agent reads.

## Considered Options

- Keep a namespace, so a future unrelated feature has somewhere to go
- Expose the verbs at the top level, and rename the vendor-shaped terms

## Decision Outcome

Chosen option: `Expose the verbs at the top level, and rename the vendor-shaped terms`.

The command is `ripwork`, and the verbs sit directly under it: `list`, `show`, `init`, `validate`,
`resolve`, `next`, `claim`, `record`, `advance`, `reclaim`, `summary`, and `conformance`. A namespace
reserved for a feature nobody has proposed is a word every user types forever.

Two renames land with it. The prose an agent reads for a step is a brief, declared as `brief:` and
kept under `briefs/`; the word it replaces names one vendor's product feature and would have made an
agent-agnostic specification read as an integration with that vendor. And every machine-readable
document is keyed `ripwork.<verb>.v1`, so a document names the program that emits it.

## Consequences

- Good: the surface reads as one program with a purpose, and nothing in it names a vendor.
- Good: a versioned document key per verb lets one verb's shape change without renaming the rest.
- Bad: an unrelated future feature has no namespace waiting for it, and adding one later is a
  breaking change to every driver and every script.
- Bad: readers arriving from the host tool this design came from will look for the old spellings and
  find nothing, and no page tells them why.

## Status

Accepted
