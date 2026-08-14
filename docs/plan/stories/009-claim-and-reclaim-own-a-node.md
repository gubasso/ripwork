# 009 — claim and reclaim own a node

## Goal

Exactly one driver owns a node at a time, and taking a node from a driver that never returned is a
recorded act rather than an edit.

## Example

Today nothing reserves work. When this story closes, ownership is a token.

```text
$ ripwork claim --run-dir /runs/42 --as vet --owner sh-1 --json
{"schema":"ripwork.claim.v1","ok":true,"as":"vet","owner":"sh-1",
 "claim_token":"<token>","dir":"/runs/42/vet","inputs":["/runs/42/draft/"]}

# sh-1 never returned
$ ripwork reclaim --run-dir /runs/42 --as vet --previous-claim <token> \
                  --owner sh-2 --reason "driver restarted" --json
{"schema":"ripwork.reclaim.v1","ok":true,"claim_token":"<new>","reason":"driver restarted"}
```

## Core

Ownership never expires. No elapsed time reclaims a node, because duration is not evidence of
anything — a driver decides, names a reason, and the record keeps it.

## In scope

- Mint a claim token and record the owner on the node.
- Store the optional `dispatch` value a driver supplies, returning it untouched and never parsing it.
- Refuse a claim on a node that is not pending.
- Reclaim against the previous token, minting a new one and appending the reason.
- Report the node's directory and its named inputs on both verbs, so a driver needs no second call.

## Out of scope

- Any lease, timeout, or expiry.
- Distributed locking. One run directory, one writer at a time.
- Resetting a node's status. Reclaim changes ownership, not outcome.

## Governed by

- `docs/reference/command-surface.md#claim` — the document and what a token is for.
- `docs/reference/run-directory.md#tokens` — the two tokens, who mints them, and how they are spent.
- `docs/decisions/ADR-0009-keep-one-writer-for-run-state.md` — why a driver never synthesizes a token.

## Amends

- `docs/reference/run-directory.md` — the tokens section describes minting that happens.

## Acceptance

- A claim mints a token and records the owner — `claim_mints_a_token`.
- A claim on a node that is not pending is invalid input — `claim_refuses_a_claimed_node`.
- A reclaim with the wrong previous token is invalid input — `reclaim_requires_the_live_token`.
- A reclaim appends its reason and leaves earlier reasons in place — `reclaims_accumulate`.
- A supplied `dispatch` value comes back byte-identical and unparsed — `dispatch_is_opaque`.
- No elapsed time changes ownership — `ownership_does_not_expire`.

## Tasks

- [ ] Implement token minting.
- [ ] Store and return the opaque `dispatch` value.
- [ ] Implement claim and its refusal cases.
- [ ] Implement reclaim, its token check, and reason accumulation.

## Rabbit holes

- A lease looks like the obvious way to recover a dead driver — escape: a driver reclaims explicitly,
  and duration is never a signal.
- Reclaim can drift into a retry — escape: it changes the owner and nothing else.

## Done when

The named tests pass unskipped and a node's ownership history is readable from the record after two
reclaims.

## Revisions

None
