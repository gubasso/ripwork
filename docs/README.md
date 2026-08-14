# Documentation

ripwork's documentation is organized by reader need. Start with the zone that matches the question,
then follow links to the single owner of each durable fact.

- [Reference](./reference/workflow-definition.md) is the specification. It is normative, and it is
  what an implementation is written against: the [workflow definition](./reference/workflow-definition.md)
  grammar and its validator rules, the [engine registry](./reference/engine-registry.md), the
  [command surface](./reference/command-surface.md), the [run directory](./reference/run-directory.md),
  the [orchestrator contract](./reference/orchestrator-contract.md), the
  [runner contract](./reference/runner-contract.md), and the [glossary](./reference/glossary.md).
- [Decisions](./decisions/README.md) answer why ripwork chose its constraints. Records freeze; the
  design they constrain does not.
- [Explanation](./explanation/coordination-model.md) holds the design forces: what ripwork, a driver,
  and a runner each own, and what the split costs.
- [Plan](./plan/README.md) states what gets built next and what bounds it.

Two zones are deliberately thin, and both for the same reason: no implementation exists yet.

`docs/explanation/` carries one page of design forces and no subsystem pages, because a subsystem page
describes software that exists and an empty scaffold readers would then trust is worse than none. Until
then, `docs/reference/` owns the design.

`docs/guides/` is absent. A guide promises sequence and completion, and there is nothing yet to
complete.
