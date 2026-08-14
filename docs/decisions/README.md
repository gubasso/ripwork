# Architecture decisions

This index answers which durable choice each lean record owns. Records preserve why ripwork chose its
constraints; the constraints themselves are stated in [reference](../reference/glossary.md), and the
design forces behind them in [coordination model](../explanation/coordination-model.md).

Every record here is `Accepted`: chosen, and nothing implemented. A record moves to `Implemented` when
it links what enacts it.

| number | title | question answered |
| ------ | ----- | ----------------- |
| 0001 | [Adopt the documentation architecture](./ADR-0001-adopt-the-documentation-architecture.md) | Where does a durable fact live, and what gates its shape? |
| 0002 | [ripwork coordinates a run and never dispatches one](./ADR-0002-ripwork-coordinates-and-never-dispatches.md) | Which of dispatch, judgment, and bookkeeping does ripwork keep? |
| 0003 | [A workflow is a directed acyclic graph of three call forms](./ADR-0003-a-workflow-is-a-graph-of-three-call-forms.md) | What may a definition express, and what is refused outright? |
| 0004 | [Select a step's engine at its definition or its direct call site](./ADR-0004-select-an-engine-at-the-definition-or-its-call-site.md) | How many writers may set a step's execution target? |
| 0005 | [Pass step artifacts by directory](./ADR-0005-pass-step-artifacts-by-directory.md) | How does a step hand what it produced to the steps that need it? |
| 0006 | [Express step exclusion with needs rather than a marker](./ADR-0006-express-exclusion-with-needs.md) | How are two steps that must not overlap kept apart? |
| 0007 | [Judge loop convergence with a prose criterion](./ADR-0007-judge-loop-convergence-with-a-prose-criterion.md) | Who decides a loop has converged, and what does ripwork still enforce? |
| 0008 | [Select a step's execution context at its node](./ADR-0008-select-an-execution-context-at-its-node.md) | Who decides whether a step crosses into a fresh context? |
| 0009 | [Keep one writer for every run-state artifact](./ADR-0009-keep-one-writer-for-run-state.md) | What makes a run recoverable after its driver dies? |
| 0010 | [Resolve the workspace per file](./ADR-0010-resolve-the-workspace-per-file.md) | How do layers combine, and what may a layer not contribute? |
| 0011 | [Make the exit protocol protocol only](./ADR-0011-make-the-exit-protocol-protocol-only.md) | What does an exit code mean, and where does terminal state live? |
| 0012 | [Expose one command whose verbs are top-level](./ADR-0012-expose-one-command-with-top-level-verbs.md) | How is the surface spelled, and which vendor-shaped words were dropped? |
| 0013 | [Preflight provider preconditions before the durable job](./ADR-0013-preflight-preconditions-before-the-durable-job.md) | How does a runner keep never launched apart from ran and failed? |

Start a new record from [template.md](./template.md). Its heading shape is gated by the `md-adr` hook,
which is why the template carries the same five sections a filled record does.
