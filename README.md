# ripwork

Run a workflow of agent steps without deciding anything.

ripwork holds the deterministic half of a multi-step agent workflow: it validates a definition,
materializes a run directory, reports which node is ready, hands out claims, writes receipts, and
gates every round boundary. It dispatches nothing. It reads no artifact an agent produced. It knows
no vendor, no model, and no account.

The other half — deciding what to run a step with, running it, and judging whether a loop has
converged — belongs to whatever drives ripwork. That driver may be a coding agent of any kind, a
continuous-integration job, or a shell script. ripwork makes no assumption about which, and the
[orchestrator contract](./docs/reference/orchestrator-contract.md) is written so that a shell script
is a conformant driver.

## What this repository is

Documentation. A specification, the decisions behind it, and the plan for building it. No
implementation exists yet, and the reference pages are written for whoever writes one.

## What ripwork will not do

- Evaluate a stopping criterion. A loop's criterion is prose a driver judges.
- Enforce what a step produced. Declared inputs and artifacts are description, checked against the
  brief at lint time and never at run time.
- Schedule, retry, or back off. A run advances because a driver called a verb.
- Store, rotate, or select a credential.
- Interpret a provider's diagnostics. What a provider said stays readable in its own vocabulary.

## Documentation

[`docs/`](./docs/README.md) is organized by reader need.

- [Coordination model](./docs/explanation/coordination-model.md) — what ripwork, a driver, and a
  runner each own, and why the split is the whole design.
- [Workflow definition](./docs/reference/workflow-definition.md) — the grammar a workflow is
  authored in, and every rule the validator applies to it.
- [Command surface](./docs/reference/command-surface.md) — every verb, its output document, and the
  exit protocol.
- [Orchestrator contract](./docs/reference/orchestrator-contract.md) — what a driver must be, must
  do, and must not do.
- [Decisions](./docs/decisions/README.md) — why ripwork is shaped this way.
- [Plan](./docs/plan/README.md) — what gets built next, and what bounds it.
