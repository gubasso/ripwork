# 014 — A loop materializes one round at a time

## Goal

A loop resolves to one node carrying a validated template, and each round appears only when a decision
asks for it.

## Example

Today a loop validates and materializes nothing. When this story closes, rounds arrive one at a time.

```text
$ ripwork resolve --run-dir /runs/51 --key implement-and-review \
                  --task-file /runs/51/task.md --json
{"ok":true,"nodes":[{"as":"implement","needs":[]},
                    {"as":"review-cycle","needs":["implement"],"kind":"loop","round":0}]}

$ ls /runs/51
implement  review-cycle  state.json  task.md

# after one advance --outcome continue
$ ls /runs/51/review-cycle
1

$ ls /runs/51/review-cycle/1
fix  review
```

## Core

A run's size is never fixed by a count nobody could know at load. The template is fully validated up
front and expanded never, so a loop that runs twice and a loop that runs six times are the same
definition and the same validation.

## In scope

- Resolve a loop to one node holding the template, the criterion, the ceiling, and a round counter.
- Materialize round 1 when the loop node becomes ready, and every later round only when a decision
  calls for it, creating each round's node directories under the loop's own.
- Freeze the loop's inputs once at entry, so a later round cannot pick up a changed value.
- Keep templates and instances separate in the record, so rounds grow without the graph growing.
- Report a loop's output directory as its last completed round's.

## Out of scope

- The decision itself and its transition rules, which are 015's.
- A second loop form, or nesting a loop inside a loop.
- Any expression over what a round produced.

## Governed by

- `docs/reference/workflow-definition.md#loops` — the loop keys, the runtime-scope rule, and the
  output-directory rule.
- `docs/reference/run-directory.md#fields-that-survive-a-restart` — why inputs are frozen at entry.
- `docs/decisions/ADR-0007-judge-loop-convergence-with-a-prose-criterion.md` — why the criterion is
  stored and never parsed.

## Amends

- `docs/reference/workflow-definition.md` — the loop section describes a materializer that exists.
- `docs/reference/run-directory.md` — the frozen inputs and the round records describe real fields.

## Acceptance

- A loop resolves to one node holding the template, with no round beyond the first — `loop_starts_unexpanded`.
- Round 1 appears on entry, and round 2 only after the decision that calls for it — `rounds_are_lazy`.
- A loop with no decision yet still has a first round to report on — `a_loop_can_start`.
- The template is validated at load, including the entries inside it — `template_validates_early`.
- A value changed after entry does not reach a later round — `inputs_are_frozen_at_entry`.
- The loop's output directory is its last completed round's — `loop_output_is_the_last_round`.

## Tasks

- [ ] Resolve a loop to one node carrying the template.
- [ ] Materialize round 1 on entry and later rounds on decision.
- [ ] Freeze inputs at entry.
- [ ] Separate templates from instances in the record.

## Rabbit holes

- Expanding the loop at resolve makes everything downstream simpler and is wrong — escape: the round
  count is unknown at load, so an expansion is a guess.
- Nested loops are a small generalization with a large state cost — escape: one level, and a second is
  a decision record.

## Done when

The named tests pass unskipped and a six-round run and a two-round run are proven to come from one
unchanged definition.

## Revisions

None
