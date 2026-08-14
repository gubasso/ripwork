# Repository Guidelines

## Scope

ripwork coordinates a workflow of agent steps. It owns validation, run materialization, readiness,
claims, receipts, and round boundaries. It owns no judgment, no dispatch, and no vendor knowledge.
The split is the design; [coordination model](./docs/explanation/coordination-model.md) explains it
and [ADR-0002](./docs/decisions/ADR-0002-ripwork-coordinates-and-never-dispatches.md) records it.

No implementation exists. This repository is a specification, the decisions behind it, and the plan
for building one.

## Vocabulary

The words below are load-bearing and are not interchangeable. Full definitions are in
[glossary](./docs/reference/glossary.md).

- ripwork is the program. There is no second name for it — no compound noun, no "the engine".
- An engine is one record naming a provider, a model, and an effort. A step declares one.
- A runner is the component that turns one step into one running agent and one set of artifacts.
- An orchestrator is any program that drives a run to a terminal state.
- A brief is the prose an agent reads for a step. Never call it a skill; that names one vendor's
  feature.
- A node is one instance in a run graph, named by its `as:` handle.

## Writing conventions

- Uppercase RFC 8174 keywords are for the reference zone, where a document is a contract an
  implementer builds against. Narrative prose uses lowercase.
- Name no vendor, product, model, binary, flag, or version outside a decision record's context
  section. A specification that names one has stopped being agent-agnostic.
- Name no implementation language, runtime, parser, or library. What ripwork must do is specified;
  how it does it is not.
- Concrete surfaces are the product and stay exact: the authored format, the verbs, the arguments,
  the output documents, the exit codes, and the run-directory layout.

## Documentation maintenance

- Put decisions in `docs/decisions/`, guides and runbooks in `docs/guides/`, exact lookup in
  `docs/reference/`, current design in `docs/explanation/`, and forward intent in `docs/plan/`.
- Keep the current design of a subsystem in one explanation page; its linked ADRs stay frozen.
- Name every ADR `ADR-<number>-<decision>.md`, keep its filled body at or below 350 words, and give
  it exactly one `Status`.
- Never delete an accepted decision. Supersede, deprecate, reject, or amend it.
- Record plan state in five lane files, keep each story in `docs/plan/stories/<id>-<slug>.md`, and
  use the optional same-name directory only for non-narrative artifacts.
- Estimate stories at `1 | 2 | 3` points of irreducible human judgment. Protect the declared core,
  cut the ordered remainder first, and split work that exceeds three points.
- Record a goal larger than one story as one epic at `docs/plan/epics/<id>-<slug>.md`, carry
  membership as `epic:` on the lane entry, and never list member stories in the epic. Stories and
  epics share one id sequence, so an id names one thing.
- Work from the current story — the topmost entry of `doing.yml`, else the topmost of `todo.yml` —
  and the individual sources its `Governed by` section names.
- Gate every fixed heading contract with one `MD043` array applied by one hook entry. Never name
  `MD043` in `.markdownlint-cli2.jsonc`, which merges over the shape and would switch it off
  silently.
- Write each durable fact once at its owning home and cross-link from everywhere else.
- Let the filesystem own state: indexes explain purpose and never replicate a directory tree.
- Keep drafts in `.draft/`, outside durable docs.
- Track perishable facts in a machine-readable registry with a cadence and a `last_checked` date.
- Use no bold or italics. Put identifiers, paths, flags, and statuses in inline code, and give every
  fenced block a language.
- Make each phase of a multi-phase guide name its input and output artifacts using upper-snake
  `<ANGLE>` tokens that never carry real values.
- Report documentation changes by ownership: which source changed, which links were added, and which
  gates passed.

The canon these rules come from is the docs-design shelf. Local exceptions:

- The plan zone follows the plan-xp method — five lane files, stories, epics, and a transition
  journal. [ADR-0001](./docs/decisions/ADR-0001-adopt-the-documentation-architecture.md) records the
  adoption.
- An explanation page arrives with the subsystem it describes. Until code exists, `docs/reference/`
  owns the design and `docs/explanation/` holds design forces only. An empty subsystem page readers
  would then trust is worse than none.
- `docs/reference/known-issues/` will exist when the first real external-system case does.
- Hand-wrap prose at 100 columns.

## Verification

- `pre-commit run --all-files` runs every documentation gate.
- Prove a heading shape fails before trusting it: add a heading the array forbids, confirm the
  failure names it, remove the heading. A shape that cannot be made to fail is not wired.
- Run no git command unless the operator explicitly authorizes it.
