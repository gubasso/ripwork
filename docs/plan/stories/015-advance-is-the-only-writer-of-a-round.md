# 015 — advance is the only writer of a round

## Goal

A driver reports its judgment of a loop's criterion, and ripwork applies or refuses the transition
against everything it can check without reading an artifact.

## Example

Today rounds materialize and nothing decides. When this story closes, the decision is a call and the
refusals are deterministic.

```text
$ ripwork advance --run-dir /runs/51 --loop review-cycle --decision-token <tok> \
                  --outcome continue --reason stalled --json
{"schema":"ripwork.advance.v1","ok":true,"loop":"review-cycle","outcome":"continue",
 "reason":"stalled","note":null,"round":3,"state":"running"}

# the round produced no files
$ ripwork advance --run-dir /runs/51 --loop review-cycle --decision-token <tok> \
                  --outcome converged --reason criterion-met --json
{"error":"illegal transition",
 "detail":"the round produced no files, so it can neither continue nor converge"}
$ echo $?
2
```

## Core

Four things are enforced without reading one artifact: the ceiling, the decision token, both closed
vocabularies, and round completeness. Everything else about the verdict is the driver's, and ripwork
records it rather than checking it.

## In scope

- Verify the decision token, and spend it on an applied transition so a replay fails.
- Refuse `continue` past the round ceiling, where reaching the ceiling is a failure rather than a
  success.
- Refuse `continue` and `converged` on a round that produced no files, while leaving `pause` and
  `abort` legal so an empty round can be escaped.
- Retain each round's outcome, reason, and note, so a convergence call stays auditable afterwards.
- Set the run's state from the outcome, with `paused` reported rather than recomputed.

## Out of scope

- Evaluating the criterion, or any part of it. It is stored and never parsed.
- Any outcome or reason outside the two closed vocabularies.
- A round argument. ripwork owns the counter and the token already binds the round.

## Governed by

- `docs/reference/command-surface.md#advance` — the two closed vocabularies and the three illegal
  transitions.
- `docs/reference/workflow-definition.md#loops` — the ceiling rule and the completeness rule.
- `docs/decisions/ADR-0007-judge-loop-convergence-with-a-prose-criterion.md` — why the driver is the
  sole verdict and what ripwork still holds.

## Amends

- `docs/reference/command-surface.md` — the advance section describes a verb that exists.
- `docs/reference/run-directory.md` — the retained per-round decision describes a stored field.

## Acceptance

- A spent decision token is refused on replay — `decision_token_is_spent_once`.
- Advancing past the ceiling is an illegal transition, and reaching it is a failure —
  `ceiling_is_a_failure`.
- `continue` and `converged` are refused on an empty round; `pause` and `abort` are not —
  `empty_round_can_only_be_escaped`.
- An outcome or reason outside its vocabulary is invalid input — `vocabularies_are_closed`.
- Each round's decision is readable afterwards with its reason — `decisions_are_retained`.

## Tasks

- [ ] Implement token verification and spending.
- [ ] Implement the three illegal transitions.
- [ ] Retain the per-round decision record.
- [ ] Set the run state, reporting `paused` rather than recomputing it.

## Rabbit holes

- A criterion this specific looks parseable — escape: it is a string, and every line of parsing is a
  second authority on the verdict.
- Round completeness can drift into reading a file to see if it counts — escape: at least one file, of
  any size, from any step in the round.

## Done when

The named tests pass unskipped and a loop reaching its ceiling is proven to end `failed` rather than
`done`.

## Revisions

None
