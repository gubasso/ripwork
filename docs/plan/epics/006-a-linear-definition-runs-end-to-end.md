# 006 — A linear definition runs end to end

## Goal

A driver takes one linear definition from resolve to a terminal state, with a durable receipt for
every node and no state it wrote itself.

## Example

Today no verb exists past validation. When this end state is reached, a driver's whole loop is four
calls repeated.

```text
$ ripwork resolve --run-dir /runs/42 --key plan-review-apply \
                  --task-file /runs/42/task.md \
                  --orchestrator '{"name":"sh","version":"1","capabilities":["subprocess"]}' --json
{"schema":"ripwork.resolve.v1","ok":true,"run_dir":"/runs/42","state":"running",
 "nodes":[{"as":"draft","needs":[]},{"as":"vet","needs":["draft"]},
          {"as":"apply","needs":["draft","vet"]}]}

$ ripwork next --run-dir /runs/42 --json
{"schema":"ripwork.next.v1","ok":true,"state":"running",
 "node":{"as":"draft","dir":"/runs/42/draft","brief":"plan","inputs":[]}}

$ ripwork claim --run-dir /runs/42 --as draft --owner sh-1 --json
{"schema":"ripwork.claim.v1","ok":true,"claim_token":"<token>","dir":"/runs/42/draft"}

# the driver dispatches, the agent writes /runs/42/draft/plan.md

$ ripwork record --run-dir /runs/42 --as draft --claim-token <token> --status done --json
{"schema":"ripwork.record.v1","ok":true,"receipt_path":"/runs/42/draft/outputs.json",
 "outputs_count":1,"run_state":"running"}
```

## Core

The driver writes no state. Every durable artifact in the run has exactly one writer, so a run that
ends is a run that can be read back from files without anyone having observed it happen.

## Out of scope

- Composites, loops, and round boundaries. A linear definition is the whole of this end state.
- Recovery after a crash. It rests on this lifecycle and comes next.
- Retries, backoff, and scheduling. A run advances because a driver called a verb.
- Any provider runner. The driver dispatches by whatever means it has.

## Governed by

- `docs/reference/command-surface.md` — every verb, its document, and the exit protocol.
- `docs/reference/run-directory.md` — the layout, the run-state document, the receipt, and the tokens.
- `docs/decisions/ADR-0009-keep-one-writer-for-run-state.md` — why one writer per artifact is what
  makes a run recoverable.
- `docs/decisions/ADR-0005-pass-step-artifacts-by-directory.md` — why a receipt inventories a
  directory instead of verifying a declaration.

## Amends

- `docs/reference/command-surface.md` — the eight verbs this end state implements stop being
  specified and start being described.
- `docs/reference/run-directory.md` — the run-state document and the receipt become artifacts a reader
  can go and look at.

## Done when

One linear definition reaches a terminal `done` from an empty directory, every node carries a receipt
whose key set is exactly the closed set, and a driver that never wrote state produced all of it.

## Revisions

None
