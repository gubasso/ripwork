# 004 — The engine registry validates

## Goal

Every registry row is held to the five invariants, so a step naming an engine that could not exist
fails before a run is materialized.

## Example

Today there is no registry to consult. When this story closes, a row that breaks an invariant is
reported with the invariant it broke.

```text
$ ripwork validate --all --json
{"schema":"ripwork.validate.v1","ok":false,"checked":[],
 "findings":[
   {"rule":"derived-id","detail":"id does not equal <provider>-<model>-<effort>"},
   {"rule":"four-fields","detail":"row carries a fifth key"},
   {"rule":"provider-effort","detail":"effort is not a rung of that provider's ladder"},
   {"rule":"declared-runner","detail":"provider declares no runner"}]}
```

## Core

Membership is the permission. An engine that cannot be run is a row that does not exist, and no flag
on a row is allowed to mean otherwise — a row plus a disabled marker is a permission the validator
would have to reason about, and reasoning is what this rule removes.

## In scope

- The five invariants: derived id, unique id, exactly four fields, a provider-valid effort, and a
  declared runner.
- Provider metadata: the runner name, the runner status, and the provider's own ordered effort ladder.
- Running the invariants on every validate call rather than behind a mode that has to be asked for.
- Exposing a lookup the grammar validator uses to decide whether an engine literal exists.

## Out of scope

- Shipping rows. Concrete providers, models, and rungs are perishable vendor data an implementation
  supplies and tracks on a cadence.
- Comparing effort across providers. Ordering is meaningful inside a ladder and undefined between
  them, and nothing here may imply otherwise.
- Letting a workspace layer contribute a row.

## Governed by

- `docs/reference/engine-registry.md` — the record, the provider metadata, and the five invariants.
- `docs/decisions/ADR-0010-resolve-the-workspace-per-file.md` — why the registry is not layered.
- `docs/decisions/ADR-0004-select-an-engine-at-the-definition-or-its-call-site.md` — where an engine
  literal may appear, which is what the lookup serves.

## Amends

- `docs/reference/engine-registry.md` — the invariants are enforced rather than specified, and the
  no-rows-ship rule states what an implementation does instead.

## Acceptance

- A row breaking each invariant is reported, one finding per invariant — `five_invariants_hold`.
- A row carrying a fifth key is refused rather than ignored — `extra_field_is_a_typo`.
- An effort absent from its provider's ladder is refused — `effort_is_on_the_ladder`.
- The invariants run on a plain validate call with no extra argument — `registry_is_never_opt_in`.

## Tasks

- [ ] Implement the record and provider-metadata readers.
- [ ] Implement the five invariants.
- [ ] Expose the engine lookup 003 consumes.
- [ ] Add a tracking entry shape for an implementation that later ships rows.

## Rabbit holes

- A disabled marker looks like a kind way to keep a row — escape: membership is the permission, and a
  row that cannot be dispatched is deleted.
- A cross-provider effort ordering looks obviously useful — escape: it is undefined, and any code that
  compares two ladders is inventing a fact.

## Done when

The named tests pass unskipped, no rows ship, and the specification states the tracking obligation an
implementation that ships rows takes on.

## Revisions

None
