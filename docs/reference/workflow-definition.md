# Workflow definition

The grammar a workflow is authored in, and every rule the validator applies to it. This page is
normative: an implementation is conformant when it accepts exactly what is described here and refuses
everything else.

The engine record a step names is owned by [engine registry](./engine-registry.md). The verbs that
read a definition, and the exit codes they report, are owned by
[command surface](./command-surface.md).

## Workspace layout

A workspace holds definitions and briefs, in four fixed members:

```text
<workspace>/
  meta.yaml     three closed defaults: context, max_rounds, max_workflow_depth
  workflows/    composite definitions; no brief: key anywhere
  steps/        leaf definitions; the only files that may carry brief:
  briefs/       the prose the agents read
```

`meta.yaml` carries exactly three keys and no others:

| Key                  | Type              | What it sets                                        |
| -------------------- | ----------------- | --------------------------------------------------- |
| `context`            | `fresh`/`inherit` | the default for a step that declares none           |
| `max_rounds`         | positive integer  | the default loop ceiling                            |
| `max_workflow_depth` | positive integer  | how deep a `workflow:` reference chain may nest     |

There is deliberately no default engine. Every step definition pins its own, so a definition read on
its own says what it will run.

The `max_rounds` default SHOULD be a single digit. A ceiling large enough to hide a runaway loop only
makes a run expensive before it fails.

## Layer resolution

Three layers resolve in this order, most specific first:

| Layer       | Root                                                              |
| ----------- | ----------------------------------------------------------------- |
| `project`   | a workspace in the working project, overridable by the environment |
| `user`      | the user's configuration directory                                 |
| `installed` | the installation's data directory                                  |

The unit of resolution is the file, not the root. One resolver walks the layers in order and returns
the first file that exists, taken whole. Nothing is merged — not two files, and not the keys inside
them — so a project-layer workflow replaces the installed one entirely rather than inheriting its
steps. `show` MUST report the resolved source per file, so shadowing stays visible rather than
inferred. [ADR-0010](../decisions/ADR-0010-resolve-the-workspace-per-file.md) owns the choice.

The engine registry is not part of this tree and is not layered. Membership in it is the permission
to dispatch.

## Call forms

A workflow's `steps:` is a list. Each entry carries exactly one kind key, and that key discriminates
leaf from composite at the call site.

| Kind key    | What it is             | Keys it takes                                 |
| ----------- | ---------------------- | --------------------------------------------- |
| `step:`     | leaf; runs one agent   | `id`, `as`, `engine`, `needs`, `context`      |
| `workflow:` | composite by reference | `id`, `as`, `needs`                           |
| `loop:`     | composite inline       | `as`, `needs`, `until`, `max_rounds`, `steps` |

A `workflow:` or `loop:` node carries no `engine` and no `context`, because it runs no agent. Its
steps each carry their own.

```yaml
id: "plan-review-apply"
steps:
  - step: {id: plan,   as: draft}
  - step: {id: review, as: vet,   needs: [draft]}
  - step: {id: apply,  as: apply, needs: [draft, vet]}
```

A step definition lives in `steps/` and carries its own keys:

```yaml
id: "review"
engine: "<provider>-<model>-<effort>"
brief: "review"
context: fresh
inputs:
  - "the upstream node directories handed in by needs:"
artifacts:
  - "review.md"
```

`inputs:` and `artifacts:` are description. They are aligned against the brief at lint time and are
never enforced at run time; see [ADR-0005](../decisions/ADR-0005-pass-step-artifacts-by-directory.md).

## Name spaces

`id:` names the definition, is unique within its directory, and resolves to a file. `as:` names this
instantiation, is unique within the graph, and is what `needs:` targets. `as:` defaults to `id:`, so
only fan-out pays for the distinction.

## Edges

`needs:` is the only edge key. It points at siblings, never crosses a scope boundary, and is never a
call. It carries both execution order and mutual exclusion: two steps that must not overlap get an
edge between them.

An edge with no corresponding data reference is an ordering constraint. Deleting it changes
behaviour while validation still passes; see
[ADR-0006](../decisions/ADR-0006-express-exclusion-with-needs.md).

An edge naming a composite resolves to every node that composite flattened into. A `workflow:`
reference expands at resolve time into the caller's own graph, each child handle prefixed with the
call's handle and separated by a dot, and the composite itself is not a node — it runs no agent and
gets no directory. So an edge written against the composite is satisfied when every one of its
children has reached `done`, and an edge may still be written against one child directly by its dotted
handle. Prefixing is what keeps every edge pointing at a sibling: after expansion, there is no scope
left to cross.

## Engines

`engine:` is REQUIRED on a step definition and OPTIONAL on a call site, where it overrides the
definition for that call only. Values are literals in both positions, never expressions. A call site
overrides only the step it calls and never reaches into a referenced workflow's interior, so the whole
precedence rule is one sentence: the direct call site wins, and nothing else sets the value.

What a valid value is, and what makes a record exist at all, is
[engine registry](./engine-registry.md).

## Execution context

`context:` is OPTIONAL on a `step:` and takes the literal `fresh` or `inherit`. Absent, the step takes
the workspace default from `meta.yaml`.

`fresh` means the driver MUST cross into a context that has not seen the run. `inherit` means it MAY
run the step in its own. The value is a literal, never an expression, and there is no call-site
override map. [ADR-0008](../decisions/ADR-0008-select-an-execution-context-at-its-node.md) owns the
choice, and [orchestrator contract](./orchestrator-contract.md) owns what a driver must do with the
value — the `inline` and `subprocess` capabilities are capabilities about exactly this key.

## Artifacts

Each step writes its artifacts into its own directory under the run directory, named by its `as:`
handle. `needs:` hands every upstream directory to the step that depends on it.

```text
<run directory>/
  draft/     plan.md writes here
  vet/       receives ../draft/
  apply/     receives ../draft/ and ../vet/
```

The exact directory layout and the receipt written into it are
[run directory](./run-directory.md).

## Loops

```yaml
- loop:
    as: review-cycle
    needs: [implement]
    until: "the reviewer reports no blocking findings; or the same findings
            recur unresolved across two consecutive rounds"
    max_rounds: 6
    steps:
      - step: {id: review, as: review}
      - step: {id: fix,    as: fix, needs: [review]}
```

`until:` and `max_rounds:` are both REQUIRED.

`until:` is a prose criterion ripwork stores and never parses. At each round boundary the driver reads
the criterion and the round's directories, then reports its decision through `advance`; ripwork
materializes the next round only on that call, one round at a time. An authored criterion SHOULD be
applicable by a reader to the round's artifacts, and SHOULD describe the stall path as well as the
success path.

Round 1 is the exception, and it is materialized when the loop node becomes ready rather than by a
decision. A decision reports on a round that happened, and there is no round 0 to report on: a loop
whose first round had to be requested by `continue` could never start, because `continue` is refused
on a round that produced no files. Entering a loop creates its first round; every round after that is
a decision.

`max_rounds:` is a hard ceiling enforced before a round is materialized, and reaching it is a failure
rather than a success. A round that produced no files refuses to continue or converge.

### The criterion and the decision are different things

`until:` and the outcome a driver reports are not two spellings of one verdict, and neither replaces
the other. `until:` is the criterion: authored once at design time and identical every round. The
outcome is the decision: emitted once per round boundary and different each round. What would be
redundant is ripwork evaluating the criterion while a driver also reported an outcome, which is two
authorities for one verdict — and removing the evaluation is what leaves the driver's report as the
sole one.

An evidence path on `until:` stays available later without breaking any authored workflow, because a
plain string remains valid if an added key is optional.

A loop is a runtime scope, not a resolve-time expansion. Composites flatten statically into dotted
sibling handles; a loop resolves to one node carrying an unexpanded but fully validated template. A
loop's output directory is its last completed round's.

## What the grammar does not have

Recorded so it is not re-proposed:

- No `if:` and no conditional of any kind, so every node in the run graph runs and the join question
  never arises.
- No `matrix:`.
- No fold operator.
- No `for_each:`. Unknown cardinality lives inside a step whose brief owns its loop.
- No exclusion marker. `needs:` carries it.
- No expression language, anywhere, in any position.
- No designated verdict step. Marking one step in a loop body as the one that decides is the same
  special-step marker `matrix:` was refused for, and it is wrong for every body where the verdict is
  not one step's to give.
- No designated output key on a loop step. It puts content in the definition and makes a step
  definition depend on its call site. The surviving form of the idea is the completeness rule, which
  asks that the round produced a file rather than that a named step produced a named one.
- No structured map for the stopping criterion, which is schema without semantics, and no machine
  predicate over a reserved file, which resurrects the expression language and disguises a judgment as
  a computation.

## Validator rules

Validation runs once at load, before any agent is spawned, so a failure does not land at depth after
real spend.

- Exactly one kind key per entry.
- `as:` unique within a graph, and every `needs:` target present in it.
- `as:` a single safe path segment, matching `[A-Za-z0-9][A-Za-z0-9._-]*`, because it names the node
  directory under the run directory.
- No cycle among `needs:` edges, and none across file references.
- `engine:` a literal present in the registry, REQUIRED on every step definition, OPTIONAL on a call
  site.
- `loop:` carrying both `until:` and `max_rounds:`.
- `id:`, `as:`, `engine:`, `context:`, and `until:` parsed as strings, which closes the scalar
  footguns a typed document format allows; `until:` also non-empty.
- `context:` a literal `fresh` or `inherit` where present, and present only on a `step:`.
- `max_rounds:` a positive integer.
- Reference nesting within `max_workflow_depth`.
- No `brief:` key anywhere under `workflows/`.

A `loop:` template is validated with the rest of the definition rather than deferred to the round that
materializes it, so the kind, engine, and scalar rules above apply to the entries inside it.

A definition whose containers are the wrong shape — a `steps:` that is not a list, a definition that
is not a mapping — MUST be reported as a finding rather than aborting the validator. One malformed
file is one finding, not the end of the run's only chance to hear about the other eleven.
