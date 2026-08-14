# 005 — validate reports every finding

## Goal

One call reports every finding a workspace produced, and its exit code says whether the workspace is
usable.

## Example

Today the rules exist and nothing emits them. When this story closes, an author fixes a workspace from
one document.

```text
$ ripwork validate --all --json
{"schema":"ripwork.validate.v1","ok":true,"action":"validate",
 "checked":["plan-review-apply","release"],"findings":[]}
$ echo $?
0
```

## Core

`ok` is false whenever a finding exists, and the exit code is `2`. A caller never has to inspect the
findings list to learn whether the workspace passed.

## In scope

- Emit the validation document with the keys checked and the findings found.
- Report `2` when any finding exists and `0` when none does.
- Accept one key, or `--all` for the whole workspace.
- Print a compact human line without `--json`, holding no stability promise.

## Out of scope

- Repairing anything. Validation stays falsifiable, and a gate that fixes what it checks can never
  fail.
- Ranking findings by severity. A finding is a finding.
- Any exit code outside the four the protocol defines.

## Governed by

- `docs/reference/command-surface.md#validate` — the document shape and what `ok` asserts.
- `docs/reference/command-surface.md#output-and-the-exit-protocol` — the four codes and what each
  describes.
- `docs/decisions/ADR-0011-make-the-exit-protocol-protocol-only.md` — why the code describes the call.

## Amends

- `docs/reference/command-surface.md` — the validate section describes a verb that exists.

## Acceptance

- A clean workspace reports `ok` true, an empty findings list, and `0` — `clean_workspace_passes`.
- A workspace with any finding reports `ok` false and `2` — `any_finding_fails`.
- `--all` checks every resolvable workflow and names each in `checked` — `all_checks_everything`.
- An unknown key is invalid input rather than an empty result — `unknown_key_is_invalid_input`.

## Tasks

- [ ] Emit the validation document over the rules from 003 and 004.
- [ ] Wire the exit protocol.
- [ ] Add the human line.

## Rabbit holes

- An autofix mode is one small step from here — escape: validation writes nothing, and a repair is a
  separate deliberate act.

## Done when

The named tests pass unskipped and the specification's validate section describes behaviour rather
than intent.

## Revisions

None
