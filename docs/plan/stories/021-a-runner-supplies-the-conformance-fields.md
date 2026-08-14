# 021 — A runner supplies the conformance fields

## Goal

One runner turns a fresh-context step into a running agent and a set of durable artifacts a caller can
read afterwards.

## Example

Today a driver dispatches by whatever means it invented. When this story closes, a runner is the means.

```text
$ ripwork-runner launch --engine <provider>-<model>-<effort> \
                        --state /runs/60/vet/job.json \
                        --brief /ws/briefs/review.md --inputs /runs/60/draft/
{"ok":true,"state":"/runs/60/vet/job.json","lifecycle":"running"}

$ ripwork-runner finalize --state /runs/60/vet/job.json --max-wall 30
{"ok":true,"lifecycle":"finalized-ok",
 "artifacts":{"output":"/runs/60/vet/final-message.txt",
              "events":"/runs/60/vet/events.log",
              "errors":"/runs/60/vet/errors.log"}}
$ echo $?
0
```

## Core

Ten conformance fields, five normalized, and not one provider diagnostic translated. A caller branches
on the exit code and reads the normalized fields; everything the provider said stays readable in its
own vocabulary.

## In scope

- Build one argument vector the caller cannot extend, and hand it to a durable job that outlives the
  calling process tree.
- Write one durable state document at a caller-named absolute path, carrying every artifact path.
- Capture the event stream whole and the error stream separately, never merged.
- Classify on finalize: done and ok, done and failed, or still running, where still running is never
  an error.
- Resolve one effort rung from the engine record, dispatching a rung a provider would silently
  substitute by omitting the argument rather than forwarding its name.

## Out of scope

- Preconditions, which are 022's and run before the state document exists.
- A second provider. One runner proves the contract.
- Mapping a provider's error taxonomy into ripwork's, rewriting its error stream, or summarizing its
  event stream.

## Governed by

- `docs/reference/runner-contract.md#conformance-fields` — the ten fields and which five are
  normalized.
- `docs/reference/runner-contract.md#the-asymmetries-the-contract-absorbs` — the five classes a runner
  is written against rather than one provider's spelling.
- `docs/reference/engine-registry.md` — the record an effort rung is resolved from.

## Amends

- `docs/reference/runner-contract.md` — the conformance fields describe a runner that exists.
- `docs/reference/engine-registry.md` — new: the first provider rows, which this runner is what
  permits to exist.

## Acceptance

- All ten conformance fields are present for a real launch — `ten_fields_are_supplied`.
- Every artifact path is absolute — `artifact_paths_are_absolute`.
- The error stream is a separate file and never merged into the event stream — `streams_stay_separate`.
- A still-running job reports `75` and is not an error — `running_is_not_a_failure`.
- The provider's diagnostics are preserved unrewritten — `diagnostics_are_not_translated`.
- A rung the provider would silently substitute is dispatched by omission — `substituted_rung_is_omitted`.

## Tasks

- [ ] Answer Q-004 before deciding who inlines a brief.
- [ ] Build the argument vector and the durable job.
- [ ] Capture the streams and write the state document.
- [ ] Implement finalize and its three-way classification.

## Rabbit holes

- A translation layer over provider diagnostics looks like a service to the caller — escape: it has to
  be corrected on every provider release, and preserving is the smaller obligation.
- A runner can drift into supervising its job — escape: it launches and returns, and the outcome comes
  from files.

## Done when

The named tests pass unskipped, Q-004 has exited, and a launch's outcome is proven reconstructable
from files alone.

## Revisions

None
