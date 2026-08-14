# 002 — The workspace resolves per file

## Goal

A workflow authored in one layer references a step shipped in another, and a reader can see which
layer every file came from.

## Example

Today nothing resolves. When this story closes, a project that shadows one workflow keeps every
shipped step it did not touch.

```text
$ ripwork show plan-review-apply --json
{"schema":"ripwork.show.v1","ok":true,"key":"plan-review-apply",
 "files":[
   {"role":"workflow","key":"plan-review-apply","layer":"project","path":"/p/ripwork/workflows/plan-review-apply.yaml"},
   {"role":"step","key":"plan","layer":"installed","path":"/usr/share/ripwork/steps/plan.yaml"},
   {"role":"step","key":"review","layer":"user","path":"/home/u/.config/ripwork/steps/review.yaml"}],
 "meta":{"context":"fresh","max_rounds":5,"max_workflow_depth":3}}
```

## Core

Nothing is merged. A file is taken whole from the first layer that has it, so a project-layer
definition replaces the shipped one entirely rather than inheriting keys nobody can see.

## In scope

- Resolve one file at a time across the layers, in a fixed order, returning the first that exists.
- Load `meta.yaml` and hold its three closed keys, refusing a fourth.
- Report the resolved layer and absolute path per file, which is what makes shadowing visible.
- List every resolvable workflow with its layer.

## Out of scope

- Validating anything the files contain. This story finds files; 003 judges them.
- Merging two files, or the keys inside one. There is no merge semantics to write.
- Letting any layer contribute a registry row, which would let a layer grant itself an engine.

## Governed by

- `docs/reference/workflow-definition.md#layer-resolution` — the layer order, and the rule that a file
  is taken whole.
- `docs/reference/workflow-definition.md#workspace-layout` — the four workspace members and the three
  closed defaults.
- `docs/decisions/ADR-0010-resolve-the-workspace-per-file.md` — why the unit is the file and why the
  registry sits outside the tree.

## Amends

- `docs/reference/workflow-definition.md` — the layer table states the roots an implementation
  actually walks.

## Acceptance

- A workflow in the project layer resolves a step present only in the installed layer —
  `resolves_across_layers`.
- A file present in two layers resolves to the more specific one, whole, with no key from the other —
  `no_merge_across_layers`.
- `show` reports a layer and an absolute path for every file it resolved — `show_reports_source`.
- A `meta.yaml` carrying a fourth key is refused — `meta_keys_are_closed`.

## Tasks

- [ ] Answer Q-001 and fix the layer count before writing the walk.
- [ ] Implement the per-file resolver over the settled layer list.
- [ ] Load and close `meta.yaml`.
- [ ] Implement `list` and `show` over the resolver.

## Rabbit holes

- Merge semantics can look like a small convenience — escape: a file is taken whole, and a partially
  inherited definition is the thing this rule exists to prevent.
- Layer roots can absorb the story into platform path conventions — escape: the roots are named in one
  place and the resolver is written against the list, not against any one platform.

## Done when

The named tests pass unskipped, Q-001 has exited, and the layer table in the specification states what
an implementation walks rather than what one might.

## Revisions

None
