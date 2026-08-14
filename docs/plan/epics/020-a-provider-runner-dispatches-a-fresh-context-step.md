# 020 — A provider runner dispatches a fresh-context step

## Goal

A step declaring `context: fresh` reaches a real agent through a runner, and everything the run needs
afterwards is readable from files the job wrote.

## Example

Today a driver dispatches by whatever means it invented. When this end state is reached, a runner is
the means, and its report says what it checked before it launched anything.

```text
$ ripwork-runner preflight --engine <provider>-<model>-<effort>
{"ok":false,"checks":[
  {"check":"program","result":"ok"},
  {"check":"version","result":"ok","floor":"<floor>"},
  {"check":"account","result":"absent","satisfied_by":"bind an account for <provider>"}]}
$ echo $?
2

# once bound
$ ripwork-runner launch --engine <provider>-<model>-<effort> \
                        --state /runs/60/vet/job.json --brief /ws/briefs/review.md
{"ok":true,"state":"/runs/60/vet/job.json","lifecycle":"running"}

$ ripwork-runner finalize --state /runs/60/vet/job.json --max-wall 30
{"ok":true,"lifecycle":"finalized-ok","output":"/runs/60/vet/final-message.txt"}
```

## Core

Ten conformance fields, five of them normalized, and not one provider diagnostic translated. A caller
branches on the exit code and reads the normalized fields; everything the provider said stays readable
in its own vocabulary in the artifacts.

## Out of scope

- Creating, storing, or rotating a credential. A runner reports a precondition and fixes none.
- A second provider. One runner proves the contract; a second proves nothing the first did not.
- Mapping a provider's error taxonomy into ripwork's, which is the layer that has to be corrected on
  every provider release.

## Governed by

- `docs/reference/runner-contract.md` — the ten conformance fields, the classification, the
  preconditions, and the five asymmetries a runner is written against.
- `docs/decisions/ADR-0013-preflight-preconditions-before-the-durable-job.md` — why preconditions are
  checked before the durable state document exists.
- `docs/reference/engine-registry.md` — the record a runner resolves an effort rung from, and the
  membership rule that decides which rows exist.

## Amends

- `docs/reference/runner-contract.md` — the conformance fields and the preflight rule stop being
  specified ahead of any implementation.
- `docs/reference/engine-registry.md` — a provider gains rows, which shipping its runner is what
  permits.

## Done when

One runner supplies all ten conformance fields for a real launch, reports every standing precondition
before creating a durable state document, and preserves the provider's own diagnostics unrewritten.

## Revisions

None
