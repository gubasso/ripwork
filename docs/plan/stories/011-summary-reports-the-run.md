# 011 — summary reports the run

## Goal

One call answers what happened in a run, without anyone reading the run-state document by hand.

## Example

Today the record exists and only a driver can interpret it. When this story closes, a person or a job
can too.

```text
$ ripwork summary --run-dir /runs/42 --json
{"schema":"ripwork.summary.v1","ok":true,"run_dir":"/runs/42","key":"plan-review-apply",
 "state":"done",
 "nodes":[{"as":"draft","status":"done","outputs_count":1,"engine":"<engine>"},
          {"as":"vet","status":"done","outputs_count":2,"engine":"<engine>"},
          {"as":"apply","status":"done","outputs_count":1,"engine":"<engine>"}]}
```

## Core

Summary reports and never gates. It is safe to call at any point in a run and changes nothing, which
is what makes it usable from a hook, a log line, or a person's terminal without thinking about it.

## In scope

- Report the run's key, its state, and one row per node with status, output count, and engine.
- Read each node's output count from its receipt, treating an absent receipt as zero rather than an
  error.
- Report `0` regardless of the run's state, because reporting a terminal failure correctly is a
  success.
- Print a compact human line without the machine-readable flag.

## Out of scope

- Any judgment about whether the run went well.
- Writing anything, including a cached rollup.
- Reading an artifact to describe it.

## Governed by

- `docs/reference/command-surface.md#summary` — the document shape.
- `docs/decisions/ADR-0011-make-the-exit-protocol-protocol-only.md` — why a terminal `failed` still
  exits `0`.

## Amends

- `docs/reference/command-surface.md` — the summary section describes a verb that exists.

## Acceptance

- A terminal `failed` run is reported at `0` with `failed` in the document — `failure_reports_at_zero`.
- A node with no receipt reports zero outputs rather than failing — `missing_receipt_counts_zero`.
- Summary writes nothing — `summary_is_read_only`.

## Tasks

- [ ] Emit the summary document over the state and the receipts.
- [ ] Add the human line.

## Rabbit holes

- A summary can grow into a dashboard — escape: one row per node, and anything richer is rendered by
  whoever wants it.

## Done when

The named tests pass unskipped and a run directory is proven unchanged across a summary call.

## Revisions

None
