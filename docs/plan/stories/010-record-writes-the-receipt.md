# 010 — record writes the receipt

## Goal

One call closes a node: it verifies the token, inventories what the node directory holds, and leaves a
receipt that says what happened.

## Example

Today a claimed node has no way to finish. When this story closes, finishing is one call and one file.

```text
$ ripwork record --run-dir /runs/42 --as vet --claim-token <token> --status done --json
{"schema":"ripwork.record.v1","ok":true,"as":"vet","status":"done",
 "receipt_path":"/runs/42/vet/outputs.json","outputs_count":2,"run_state":"running"}

$ cat /runs/42/vet/outputs.json
{"schema":"ripwork.receipt.v1","as":"vet","status":"done",
 "inputs":["draft/"],"outputs":[["review.md",1841],["final-message.txt",302]],
 "dispatch":"<opaque>","engine":{"id":"<engine>"},"owner":"sh-1","error":null}
```

## Core

The key set is closed and enforced by equality on every write. Presence tests would let a token count,
a duration, or a cost be added quietly later, and the whole value of a closed receipt is that nobody
can.

## In scope

- Verify the claim token and refuse a stale or missing one.
- Inventory the node directory, and only it — the receipt never walks above the directory it describes.
- Carry the `dispatch` value the claim supplied, unparsed.
- Write the receipt, then update run state and recompute the run's own status.
- Carry an error object on a failed node, so a failure still leaves a receipt.

## Out of scope

- Reading any artifact. Files are counted and sized, never opened.
- Verifying what the step declared it would produce. That alignment is lint-time work on the brief.
- Moving a node out of `failed`, which is Q-002's to settle.

## Governed by

- `docs/reference/run-directory.md#the-receipt` — the closed key set and the equality rule.
- `docs/reference/command-surface.md#record` — the arguments and the document.
- `docs/decisions/ADR-0005-pass-step-artifacts-by-directory.md` — why a receipt inventories rather
  than verifies.

## Amends

- `docs/reference/run-directory.md` — the receipt section describes a file that gets written.

## Acceptance

- A stale or missing claim token is invalid input — `receipt_requires_the_live_token`.
- The receipt's key set is exactly the closed set, checked by equality — `receipt_keys_are_closed`.
- A failed node still gets a receipt carrying an error object — `failure_still_leaves_a_receipt`.
- The inventory covers the node directory and nothing above it — `receipt_sees_one_directory`.
- No file in the node directory is opened; the inventory is names and sizes — `inventory_never_reads`.
- A malformed receipt is an internal failure, not invalid input — `emitter_faults_are_internal`.

## Tasks

- [ ] Answer Q-002 before deciding what a `failed` status commits the run to.
- [ ] Implement the inventory as names and sizes only.
- [ ] Implement the closed-key-set equality check on every write.
- [ ] Update state and recompute the run status.

## Rabbit holes

- A receipt can grow into telemetry one useful field at a time — escape: the equality check makes each
  addition a visible edit to the specification.
- Counting can become reading — escape: names and sizes only, and the agent's final message stays a
  file the inventory names rather than a key the receipt carries.

## Done when

The named tests pass unskipped, Q-002 has exited, and adding a field to the receipt is proven to fail
the equality check.

## Revisions

None
