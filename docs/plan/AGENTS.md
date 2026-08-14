---
digest-of: docs/plan
last-synced: 2026-08-14
token-estimate: 1400
---

# Plan

How ripwork plans. The topmost entry of `lanes/doing.yml` — or of `lanes/todo.yml` when that lane is
empty — is the current story.

Everything needed to write a story, move an entry, and run the gate is in this file. The method is
plan-xp; this digest carries it so a session needs no checkout of it.

## What this directory is

The record. Five lane files, one document per story, one per epic, a charter, and a list of open
questions.

Lane membership is the workflow state. There is no status field and no priority field, because
membership is the filename and ranking is the sequence position inside it — a second store of either
would be a fact that can disagree with itself.

| Lane          | What membership means                        |
| ------------- | -------------------------------------------- |
| `backlog.yml` | agreed, not scheduled                        |
| `todo.yml`    | scheduled; the topmost entry is what to start |
| `doing.yml`   | in flight                                    |
| `review.yml`  | done and awaiting judgment                   |
| `closed.yml`  | finished, append-only, ordered by close date |

All five files exist even when a lane is empty. A missing lane file is a failure, not an empty lane.

Nothing derived is stored. Boards, graphs, and rollups are rendered on demand and never committed.

## Writing a story

One story per file, `stories/<id>-<slug>.md`. The id is three digits and the slug is lowercase
kebab-case, and both must match the lane entry in both directions.

The heading sequence is fixed and gated by the `md-story` hook. Every heading, in this order:

```text
## Goal
## Example
## Core
## In scope
## Out of scope
## Governed by
## Amends
## Acceptance
## Tasks
## Rabbit holes
## Done when
## Revisions
```

An epic is `epics/<id>-<slug>.md`, gated by `md-epic`, and has its own sequence:

```text
## Goal
## Example
## Core
## Out of scope
## Governed by
## Amends
## Done when
## Revisions
```

Four story headings are absent from an epic on purpose. `In scope` orders a remainder against a point
budget and an epic has no budget; `Acceptance`, `Tasks`, and `Rabbit holes` are obligations on a work
session, and no session implements an epic.

What each heading is for, in one line: `Goal` is the outcome in a sentence; `Example` shows it
happening; `Core` is the one guarantee never cut; `In scope` and `Out of scope` are the boundaries,
and the second is where a rejected option goes with its reason; `Governed by` and `Amends` are the two
document directions below; `Acceptance` states the behaviour and names the tests that prove it;
`Tasks` is the checklist; `Rabbit holes` names each way the work could sprawl and its escape;
`Done when` is the condition for closing; `Revisions` records changes to the agreement made after the
work began, and is `None` until one happens.

A story and a spike need a fenced block under `Example`. A chore does not, because it has no
customer-visible outcome to show.

Write `None` rather than leaving a section empty. An empty section is indistinguishable from one
nobody filled in.

Ids are unique across stories and epics alike, so one id names one thing anywhere in this directory.
An epic never gets a lane entry; membership runs one way, from a story's `epic:` field to the
document, and the document names no members.

## The lane entry

```yaml
lane: todo
stories:
  - id: "009"
    slug: a-slug-matching-the-filename
    type: story
    points: 2
    summary: "One double-quoted line of 60 to 400 characters saying what the work is, not what it is called."
    needs: ["006"]
    epic: "001"
    tags: [runtime]
    note: waits on the gate landing first
```

`id`, `slug`, `type`, `points`, and `summary` are required. `type` is `story`, `spike`, or `chore`.
`summary` says what the work is; `note` is optional and says how the entry is being handled instead.
In `closed.yml` and nowhere else, `outcome` is `done`, `cut`, or `reshaped`, `closed` is an ISO date,
and `reshaped` also requires `succeeded_by`.

Points are units of irreducible human judgment, not time: `1` confirmation only, `2` one judgment, `3`
two judgments or one hard to reverse. There is no `4` — split the story along the judgments its
`Acceptance` already names.

The record is parsed by a flat canonical subset: flat mappings, one entry nesting level, flow
sequences on one line for `needs` and `tags`. Anchors, aliases, block scalars, and multi-document
documents are rejected. Quote the id, because an unquoted `007` is octal 7 under the older spec, and
quote the summary, because an unquoted scalar carrying a colon is invalid.

## Ranking

An entry is eligible when every id in its `needs` is in `closed.yml` and no open question blocks it.

Two rules, both gated:

- R1, in `todo.yml` only: every eligible entry sits above every ineligible one. This is what makes the
  topmost entry a head read rather than a search.
- R2, in `backlog.yml` and `todo.yml`: no entry sits above an entry it needs. Only same-lane edges
  count; a dependency in another lane is not a position.

A blocking question lives in `open-questions.md`, not on the entry:

```text
Blocks: 004, 005 — the storage choice changes both interfaces
```

A comma-separated id list, then an optional em dash and a reason. Every named id must exist, none may
already be in `doing` or `review`, and at least one must still be open — a question whose every target
has closed is stale.

The rank repair is run explicitly and never from a hook. Validation has to stay falsifiable: a gate
that repairs what it is checking can never fail.

## Moving an entry

A lane change is an act the program performs, never a file edit. An edit changes the state and records
nothing: where an entry sits is only half of what a record can say, and when it arrived is the other
half that no lane file holds. The verb writes both, appends the event to `journal/<id>.tsv`, and
repairs the ranking — all of it, or none of it and a message naming the check that stopped it.

Closing adds two fields and only then: `outcome` and `closed`. `done` delivered the goal, `cut`
delivered nothing, and `reshaped` was replaced by another entry that `succeeded_by` names.

An entry may not sit in `doing` or `review` while anything it needs is still open, or while an open
question names it.

The journal directory appears with the first transition. An entry that has never moved has no file,
which is honest rather than a file holding nothing.

## Governed by, and Amends

`Governed by` is inbound: the individual sources the work session must load before it starts. State
the claim each one establishes, so a reader knows what it contributes without opening it.

`Amends` is outbound, and it is a promise: the documents this work must leave changed. Every path is
relative to the repository root and must resolve. Only the leading inline-code token of a list item is
treated as a path, so an assertion may mention a flag in inline code freely. A document the work
creates opens its assertion with `new:` and is not required to exist.

The part that is easy to skip: in the same change as the behaviour, the story's `Acceptance`
assertions are rewritten in the present tense into the amended document. The story keeps its original
acceptance as the agreement that was made; the amended document says what is true now. No tool can see
whether that transfer happened.

For ripwork this is where most of the work lands. The specification under `docs/reference/` is written
ahead of the code, so a story's `Amends` usually names the page whose forward-looking wording it makes
current, rather than a page it invents.
