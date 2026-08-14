# 016 — A crashed run reconciles

## Goal

A driver returning after a crash learns exactly which work never terminated, and dispatches nothing
twice.

## Example

Today a killed driver leaves a run whose claimed nodes are indistinguishable from running ones. When
this story closes, the record says which is which.

```text
$ ripwork next --run-dir /runs/42 --json
{"schema":"ripwork.next.v1","ok":false,"state":"running","node":null,
 "reason":"vet is claimed by sh-1 with no receipt","lease_age":"<age>"}
$ echo $?
75

$ ripwork reclaim --run-dir /runs/42 --as vet --previous-claim <old> \
                  --owner sh-2 --reason "driver restarted" --json
{"ok":true,"claim_token":"<new>"}
```

## Core

An absent receipt means the node never terminated. That is the only inference the record supports, and
it is enough: a claimed node with a receipt is finished, a claimed node without one is not, and no
third state has to be guessed at.

## In scope

- Report a claimed node with no receipt as unterminated, with its lease age as information rather than
  as a threshold.
- Recheck the definition digest on every call and fail closed on a mismatch.
- Verify the record's halves agree — a node's status and the artifacts under its directory — and name
  both sides of any disagreement.
- Hand the `dispatch` value back on `next`, so a returning driver can ask its own runtime what became
  of the work before deciding between recording an outcome and reclaiming.

## Out of scope

- Deciding anything from elapsed time. The lease age is reported and never compared.
- Retrying, or moving a node out of `failed`.
- Any recovery the driver does not initiate.

## Governed by

- `docs/reference/run-directory.md#crash-semantics` — what an absent receipt means and what a driver
  does about it.
- `docs/reference/run-directory.md#fields-that-survive-a-restart` — the digest and the opaque
  driver-supplied value.
- `docs/decisions/ADR-0009-keep-one-writer-for-run-state.md` — why reconstruction from files is the
  whole recovery story.

## Amends

- `docs/reference/run-directory.md` — the crash semantics describe behaviour rather than intent.

## Acceptance

- A claimed node with no receipt is reported unterminated, never lost and never done —
  `absent_receipt_means_unterminated`.
- A kill in every lifecycle phase leaves a run that reaches a terminal state after one reclaim each —
  `every_phase_recovers`.
- A driver that reconciles through `dispatch` before reclaiming dispatches no node twice —
  `reconciled_restart_does_not_double_dispatch`.
- A definition edited under a live run fails closed on the digest — `digest_fails_closed`.
- The `dispatch` value survives the crash and is returned on `next` — `dispatch_survives_a_crash`.

## Tasks

- [ ] Answer Q-002 alongside 010, since a failed node's path decides what recovery can offer.
- [ ] Report unterminated nodes with a lease age and their `dispatch` value.
- [ ] Recheck the digest on every call.
- [ ] Cover a kill in each lifecycle phase.

## Rabbit holes

- A lease age invites a timeout — escape: it is reported for a person to read and compared by nothing.
- Recovery can grow into supervision — escape: ripwork observes no process, so it cannot tell a node
  that never ran from one that finished before the crash, and the driver is the only party that can.

## Done when

The named tests pass unskipped, Q-002 has exited, and a kill in each phase is proven to yield exactly
one dispatch per node.

## Revisions

None
