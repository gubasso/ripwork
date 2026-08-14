# 003 — The definition grammar validates

## Goal

Every structural rule in the grammar is applied to a loaded definition, and a definition breaking four
of them reports four.

## Example

Today a malformed definition is indistinguishable from a good one. When this story closes, each
violation names its rule.

```text
$ ripwork validate plan-review-apply --json
{"schema":"ripwork.validate.v1","ok":false,"checked":["plan-review-apply"],
 "findings":[
   {"rule":"one-kind-key","detail":"entry 2 carries both step: and workflow:"},
   {"rule":"unique-handle","detail":"as: vet declared twice"},
   {"rule":"known-edge","detail":"needs: [drafts] targets no handle in this graph"},
   {"rule":"safe-handle","detail":"as: ../escape is not a single safe path segment"}]}
```

## Core

One malformed file is one finding. A container of the wrong shape is reported and the validator keeps
going, because aborting on the first fault hides the other eleven and teaches an author to fix
definitions one call at a time.

## In scope

- Exactly one kind key per entry, and only the keys that kind takes.
- Handles unique within a graph, every edge target present in it, and no cycle among edges or across
  file references.
- A handle matching a single safe path segment, because it names a directory under the run directory.
- The scalar rules: the string-typed keys parsed as strings, a non-empty criterion, a positive round
  ceiling, a context that is one of the two literals and present only on a leaf.
- Reference nesting within the workspace depth ceiling, and no brief key anywhere under `workflows/`.
- A loop template validated with the rest of the definition rather than deferred to the round that
  would materialize it.

## Out of scope

- Checking that an engine literal exists. That is 004's registry, and this story consumes it.
- Aligning a step file's declared inputs and artifacts against its brief. Real work, and it belongs
  with the brief.
- Any check that requires reading a run directory. Nothing is materialized here.

## Governed by

- `docs/reference/workflow-definition.md#validator-rules` — the rule set, in full.
- `docs/reference/workflow-definition.md#call-forms` — the three kinds and the keys each one takes.
- `docs/decisions/ADR-0003-a-workflow-is-a-graph-of-three-call-forms.md` — why the grammar refuses a
  conditional, so no rule here has a branch to consider.

## Amends

- `docs/reference/workflow-definition.md` — the validator rule set describes rules that run.

## Acceptance

- A definition breaking four distinct rules reports four findings in one call — `findings_are_plural`.
- A handle that is not a single safe path segment is refused before any directory could be named after
  it — `handle_is_a_safe_segment`.
- A cycle across two referenced files is found, not only a cycle inside one — `cycle_across_files`.
- A definition whose `steps:` is not a list is a finding, not an abort — `malformed_container_is_a_finding`.
- Entries inside a loop template are held to the kind and scalar rules — `loop_template_is_validated`.

## Tasks

- [ ] Implement the rule set against the resolver from 002.
- [ ] Implement finding accumulation so no rule short-circuits the rest.
- [ ] Cover each rule with a definition that breaks it and nothing else.

## Rabbit holes

- A rule set can grow toward a schema language — escape: implement exactly the rules the specification
  lists, and route a proposed new one through an amendment first.
- Cycle detection can expand into a general graph library — escape: the graph is small and bounded by
  the depth ceiling.

## Done when

The named tests pass unskipped and every rule in the specification has a test that fails when the rule
is removed.

## Revisions

None
