# ADR-0001: Adopt the documentation architecture

## Context and Problem Statement

ripwork starts as a specification with no code, and its design arrived as a withdrawn draft whose
records had been accepted, contradicted, and then removed. A repository in that position needs one
rule about where a durable fact lives before it writes any of them, or the same drift repeats: a
contract stated in three places, a decision readable only as a changed page, and a plan that cannot
be told apart from an exploration.

## Considered Options

- Organize by topic, one directory per subsystem
- Adopt reader-need zones and invent a plan format
- Adopt reader-need zones and the plan-xp plan method

## Decision Outcome

Chosen option: `Adopt reader-need zones and the plan-xp plan method` — the zones give every durable
fact one owner, and the plan method is already specified and gated rather than something ripwork
would have to design while it has no product yet.

Decisions go in `docs/decisions/`, guides in `docs/guides/`, exact lookup in `docs/reference/`,
current design in `docs/explanation/`, and forward intent in `docs/plan/`. An ADR is
`ADR-<number>-<decision>.md`, at or below 350 words, with one `Status` from the closed lifecycle, and
is never deleted once past `Proposed`. Every fixed heading shape is one `MD043` array applied by one
hook entry, and the project config names `MD043` at no value.

Two local exceptions follow from having no code. An explanation page arrives with the subsystem it
describes, so `docs/reference/` owns the design until then and `docs/explanation/` holds design
forces only. And `docs/reference/known-issues/` waits for the first real external-system case.

## Consequences

- Good: a reader locates a fact by the question they arrived with, and a non-owner links instead of
  restating.
- Good: the plan record is gated by a linter rather than by review habit.
- Bad: the plan gate needs an installed plan-xp, so until one exists the lane files are held by
  review.
- Bad: an empty explanation zone reads as an omission to anyone who has not read this record.

## Status

Accepted
