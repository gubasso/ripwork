# 022 — A runner preflights its preconditions

## Goal

A launch that could never have worked is reported before anything is spent, and stays
distinguishable from a run that happened and failed.

## Example

Today a missing credential surfaces as an agent's own failure, if at all. When this story closes, it
surfaces before a job exists.

```text
$ ripwork-runner preflight --engine <provider>-<model>-<effort> --json
{"ok":false,"checks":[
  {"check":"program","result":"ok"},
  {"check":"version","result":"ok","floor":"<floor>"},
  {"check":"account","result":"absent","satisfied_by":"bind an account for this provider"},
  {"check":"credential","result":"skipped","detail":"no account to check one against"}]}
$ echo $?
2

$ ls /runs/60/vet
# empty: no durable state document was created
```

## Core

The check runs before the durable state document exists. Once a job exists its status belongs to the
agent, so a precondition caught afterwards is a precondition indistinguishable from a refusal the
agent produced.

## In scope

- Check the five standing preconditions a retry reproduces: the provider's program, a version floor,
  an account binding, a profile resolution, and a credential.
- Report each with what was checked and what would satisfy it.
- Run the check on launch as well as on demand, so a caller cannot skip it by not asking.
- Create nothing when a check fails.

## Out of scope

- Creating, storing, rotating, or selecting a credential. A runner reports and fixes nothing.
- Parsing a provider's error prose to recognize a failed launch, which makes correctness depend on
  strings a release can reword.
- Catching a precondition that only appears mid-run. Preflight narrows the window; it does not close
  it.

## Governed by

- `docs/reference/runner-contract.md#preconditions` — the five standing preconditions and the
  reporting shape.
- `docs/decisions/ADR-0013-preflight-preconditions-before-the-durable-job.md` — why the check precedes
  the state document.
- `docs/reference/runner-contract.md#the-asymmetries-the-contract-absorbs` — why a launcher that
  replaces itself with its child leaves nothing to check afterwards.

## Amends

- `docs/reference/runner-contract.md` — the preconditions section describes a check that runs.

## Acceptance

- A failing precondition creates no durable state document — `failed_preflight_creates_nothing`.
- Each of the five preconditions is reported with what would satisfy it — `checks_name_their_remedy`.
- Preflight runs on launch and not only on demand — `preflight_is_not_opt_in`.
- Never launched and ran and failed are distinguishable without reading provider prose —
  `launch_failure_is_distinguishable`.

## Tasks

- [ ] Implement the five checks.
- [ ] Order preflight before the state document is created.
- [ ] Wire preflight into launch.

## Rabbit holes

- Authentication work expands without limit — escape: stop at reporting, and name nothing a runner
  would have to fix.
- Provider prose looks like a cheap way to classify — escape: a provider rewords it and the runner
  breaks silently.

## Done when

The named tests pass unskipped and a failed preflight is proven to leave an empty node directory.

## Revisions

None
