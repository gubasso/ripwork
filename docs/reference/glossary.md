# Glossary

Every term ripwork's specification uses in a narrow sense, resolved to the page that owns it. A term
whose ordinary English meaning is the intended one is not listed.

## The parties

- ripwork — the program. There is no second name for it and no compound noun; not "the engine", not
  "the runtime". It coordinates a run and dispatches nothing;
  [ADR-0002](../decisions/ADR-0002-ripwork-coordinates-and-never-dispatches.md).
- Orchestrator, or driver — any program that drives a run to a terminal state. The two words are
  interchangeable and both are used. What one owes ripwork is
  [orchestrator contract](./orchestrator-contract.md).
- Runner — the component that turns one fresh-context step into one running agent and one set of
  durable artifacts. A launcher, not a supervisor; [runner contract](./runner-contract.md).
- Provider — the vendor whose agent a runner launches. Named nowhere in this specification.
- Agent — whatever a runner launches to perform one step. ripwork never observes it and never reads
  what it produced.

## The definition

- Workspace — the four-member tree holding definitions and briefs, resolved across three layers;
  [workflow definition](./workflow-definition.md#workspace-layout).
- Workflow — a definition under `workflows/`. A list of entries, each carrying exactly one kind key.
- Step definition — a definition under `steps/`. The only file that may carry `brief:`, and the only
  one that pins an engine of its own.
- Brief — the prose an agent reads for a step, under `briefs/`. Never called a skill; that names one
  vendor's feature. The brief owns what the step reads and produces;
  [ADR-0005](../decisions/ADR-0005-pass-step-artifacts-by-directory.md).
- Leaf — a `step:` entry. It runs one agent.
- Composite — a `workflow:` or `loop:` entry. It runs no agent, so it carries no engine and no
  context.
- Kind key — the one key discriminating leaf from composite at a call site: `step:`, `workflow:`, or
  `loop:`.
- Edge — a `needs:` entry. The only edge directive, carrying both order and mutual exclusion;
  [ADR-0006](../decisions/ADR-0006-express-exclusion-with-needs.md).
- Engine — one registry record naming a provider, a model, and an effort. Not a component and not a
  program; [engine registry](./engine-registry.md).
- Effort ladder — a provider's own ordered list of effort rungs. Ordering is meaningful inside a
  provider and undefined across providers.
- Context — whether a step runs in a context that has not seen the run, `fresh`, or may run in the
  driver's own, `inherit`;
  [ADR-0008](../decisions/ADR-0008-select-an-execution-context-at-its-node.md).

## The run

- Run — one materialized execution of one definition, rooted at a run directory.
- Node — one instance in the run graph. Every node has a handle, a kind, and a status.
- Handle — a node's `as:` value. Unique in the run graph, a single safe path segment, the target of
  every `needs:`, and the name of the node's directory. A composite's children flatten into dotted
  sibling handles.
- Identifier — a definition's `id:` value. Unique within its directory, and what resolves to a file.
  Distinct from a handle: `id:` names the definition and `as:` names this instantiation.
- Node directory — the directory a node writes into, named by its handle. `needs:` hands every
  upstream node directory to the node that depends on it.
- Frontier — the set of nodes ready to dispatch. `next` reports at most one of them absent the
  `parallel` capability.
- Ready — pending, with every `needs:` target at `done`.
- Claim — ownership of one node, taken with `claim` and carrying a token. Ownership never expires;
  [ADR-0009](../decisions/ADR-0009-keep-one-writer-for-run-state.md).
- Reclaim — taking a node whose previous owner did not return. Requires the previous token and a
  recorded reason.
- Dispatch value — the opaque value a driver supplies at `claim` and reads back on `next`. It is how a
  returning driver tells work that never ran from work that finished before the crash; ripwork stores
  it and never parses it.
- Receipt — the inventory of a node directory that `record` writes. A closed key set holding names
  and sizes; ripwork counts files and never opens them;
  [run directory](./run-directory.md#the-receipt).
- Round — one iteration of a loop, materialized one at a time. The first arrives when the loop node
  becomes ready; every later one is written only by `advance`.
- Decision token — the token binding one round boundary. Spent by an applied transition, so it cannot
  be replayed.
- Complete round — a round in which at least one step produced at least one file. An incomplete round
  refuses `continue` and `converged`.
- Terminal state — `done` or `failed`. It lives in a document's state field, never in an exit code;
  [ADR-0011](../decisions/ADR-0011-make-the-exit-protocol-protocol-only.md).

## The contract

- Capability — one member of the closed set a driver declares at `resolve`: `parallel`, `inline`,
  `ask-user`, or `subprocess`. ripwork enforces rather than trusts.
- Conformant driver — a program satisfying every obligation and no prohibition in
  [orchestrator contract](./orchestrator-contract.md). A shell script qualifies, deliberately.
- Conformance field — one of the ten a runner supplies. Five are normalized and five are
  provider-owned; [runner contract](./runner-contract.md#conformance-fields).
- Membership is the permission — the registry rule that an engine which cannot be run is a row that
  does not exist, rather than a row carrying a false flag.
- Single writer — the rule that each durable artifact has exactly one writer. The agent owns its node
  directory, ripwork owns run state and receipts, and a driver owns neither.
