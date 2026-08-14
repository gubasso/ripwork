# 001 — A definition validates before anything runs

## Goal

An author learns every structural mistake in a workspace from one call, before any run directory
exists and before anything has been spent.

## Example

Today there is nothing to call. When this end state is reached, a workspace with four mistakes in it
reports four.

```text
$ ripwork validate --all --json
{"schema":"ripwork.validate.v1","ok":false,"action":"validate",
 "checked":["plan-review-apply","release"],
 "findings":[
   {"key":"plan-review-apply","rule":"unique-handle","detail":"as: vet declared twice"},
   {"key":"plan-review-apply","rule":"known-engine","detail":"engine: no such record"},
   {"key":"release","rule":"one-kind-key","detail":"entry carries both step: and loop:"},
   {"key":"release","rule":"loop-keys","detail":"loop: review-cycle has no max_rounds"}]}
$ echo $?
2
```

## Core

Validation is complete before it is convenient. Every rule in the specification runs on every call,
and a definition with several faults reports all of them rather than the first — an author who has to
rediscover mistakes one call at a time will stop trusting the gate and start reading the parser.

## Out of scope

- Materializing anything. This end state creates no run directory and writes no state.
- Checking a step file's declared inputs and artifacts against its brief. That alignment is real work
  and belongs with the brief, not with the grammar.
- Shipping registry rows. The invariants are gated here; the rows are vendor data an implementation
  supplies and tracks.

## Governed by

- `docs/reference/workflow-definition.md` — the grammar, and the validator rule set every member
  implements.
- `docs/reference/engine-registry.md` — the engine record and the five invariants every row satisfies.
- `docs/decisions/ADR-0003-a-workflow-is-a-graph-of-three-call-forms.md` — why the grammar admits
  three call forms and no conditional.
- `docs/decisions/ADR-0010-resolve-the-workspace-per-file.md` — why resolution is per file and why
  the registry sits outside the layered tree.

## Amends

- `docs/reference/workflow-definition.md` — the validator rules stop describing what an implementation
  will check and state what it checks.
- `docs/reference/engine-registry.md` — the five invariants become enforced rather than specified.

## Done when

A workspace containing a deliberate instance of every rule in the specification reports one finding
per instance, in one call, and creates nothing on disk.

## Revisions

None
