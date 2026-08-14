# Run directory

The durable record of one run: its layout, the run-state document, the receipt, and the fields that
make a crashed run recoverable. This page is normative.

Everything here has one writer. The agent owns its own node directory; ripwork owns the run-state
document and every receipt; a driver owns neither.
[ADR-0009](../decisions/ADR-0009-keep-one-writer-for-run-state.md) owns the rule, and the
[orchestrator contract](./orchestrator-contract.md) states it as a prohibition.

## Layout

`resolve` validates first and materializes second, so an invalid definition leaves no directories
behind. Each node then writes into its own directory, and `needs:` hands every upstream directory to
the node that depends on it.

```text
<run directory>/
  state.json      the whole run: nodes, statuses, tokens, engines
  draft/          the plan node's artifacts
    outputs.json  its receipt, written by ripwork
  vet/            receives ../draft/
    outputs.json
  apply/          receives ../draft/ and ../vet/
    outputs.json
```

A node directory is named by the node's `as:` handle, which is why `as:` MUST be a single safe path
segment. A composite runs no agent and produces no directory of its own beyond what its children
wrote.

Every path ripwork reports is absolute. A relative path resolves against a working directory that a
dispatched agent's is not the caller's to assume.

## The run-state document

One document, keyed `ripwork.state.v1`, written atomically. A partial state document is worse than
none, and reading or persisting it is the only thing that reports an internal failure.

| Field              | What it holds                                                   |
| ------------------ | --------------------------------------------------------------- |
| `schema`           | `ripwork.state.v1`                                               |
| `key`              | the definition this run resolved                                 |
| `run_dir`          | the absolute run directory                                       |
| `task_file`        | the absolute task file the run was given                         |
| `state`            | `running`, `paused`, `done`, or `failed`                         |
| `max_fresh_depth`  | the driver's declared depth ceiling, or absent                   |
| `orchestrator`     | the driver's declaration, stored as given                        |
| `meta`             | the workspace defaults in effect for this run                    |
| `workflow_digest`  | a digest of the resolved definition                              |
| `nodes`            | one record per node in the run graph                             |

`workflow_digest` is rechecked on every call and fails closed on mismatch. A definition edited under a
live run is a definition whose run state no longer describes what will happen.

## A node record

| Field         | What it holds                                                       |
| ------------- | ------------------------------------------------------------------- |
| `as`          | the handle; unique in the run graph                                  |
| `id`          | the definition this node instantiates                                |
| `kind`        | `step`, `workflow`, or `loop`                                        |
| `needs`       | the handles this node waits on                                       |
| `engine`      | the resolved engine record, or null for a composite                  |
| `brief`       | the brief this step's agent reads, or null for a composite           |
| `dir`         | the node directory, relative to the run directory                    |
| `status`      | `pending`, `claimed`, `done`, or `failed`                            |
| `claim_token` | the live claim, or null                                              |
| `owner`       | who holds the live claim, or null                                    |
| `reclaims`    | the reason given for each reclaim, in order                          |

A `loop` node carries seven more: `round`, `max_rounds`, `round_produced_files`, `decision_token`, and
the last reported `outcome`, `reason`, and `note`.

Templates and instances stay separate top-level maps, so rounds grow without the graph growing.

## The receipt

One `record` call inventories the node directory, writes `<as>/outputs.json`, and updates the run
state.

The receipt lists what the directory holds rather than verifying it against a declaration. ripwork
counts files and never reads them.

```text
ripwork.receipt.v1 — keys, closed and sorted:
  as, dispatch, engine, error, inputs, outputs, owner, schema, status
```

The key set is closed to inputs, outputs, state, and dispatch identity. No token counts, no cost, no
durations, no provider response metadata, and no content from any file. An implementation MUST enforce
the closed key set as an equality check on every write, not as a set of presence tests: weakening it
to presence would remove the only thing stopping a token count from being added quietly.

The agent's final message carries no key of its own. It is a file the runner wrote into the node
directory, so the inventory already names it, and copying its bytes into the receipt would make
ripwork the one thing that reads what an agent produced. A driver wanting the text opens the file.

A failed node still gets a receipt, carrying an `error` object with its reason. A malformed receipt is
ripwork's own emitter failing, so it reports an internal failure rather than an invalid input.

## Tokens

Two tokens, each minted by ripwork and each spent once.

A claim token is minted by `claim` and by `reclaim`, and is the only thing `record` accepts for that
node. A decision token binds one round boundary and is the only thing `advance` accepts for that loop.
An applied transition spends its decision token, so replaying it MUST fail as an invalid input.

A driver never synthesizes a token, a handle, an id, or a counter. Every one of them is minted here.

## Fields that survive a restart

Four fields do work out of proportion to their size, and each exists for one recovery case.

| Field                | Why it exists                                                          |
| -------------------- | ---------------------------------------------------------------------- |
| `inputs_frozen`      | materialized once at loop entry and never re-read, so a round cannot pick up a changed value |
| `rounds[].decision`  | retains each round's outcome, reason, and note, so a convergence call stays auditable after the fact |
| `dispatch`           | an opaque driver-supplied value ripwork stores and hands back without interpreting |
| `workflow_digest`    | rechecked on every call, failing closed on mismatch                     |

`dispatch` is what keeps the contract vendor-neutral across a crash. A driver supplies it at `claim`,
storing whatever it needs to find its own work again — a process handle, a conversation identifier, a
job name — and ripwork returns it on `next` and in the receipt, untouched, having never parsed it.

## Crash semantics

An absent receipt means the node never terminated, and that is the only inference ripwork can make. It
deliberately does not mean the work never ran: a driver killed between an agent finishing and `record`
being called leaves exactly the same record as one killed before it dispatched.

Resolving that ambiguity is the driver's, and `dispatch` is what it resolves it with. On restart the
driver reads the value it supplied at `claim`, asks its own runtime what became of it, and then either
records the outcome it finds or reclaims and dispatches again. A driver that reclaims without that
check can dispatch a node twice, and no ripwork verb can prevent it — ripwork observes no process, so
it cannot know what the driver's runtime did.

Duration is never a failure signal. A node that has been claimed for a long time is a node that has
been claimed for a long time; only a driver's explicit reclaim moves it.

No node moves out of `failed`. Such a node is blocked, and so is everything downstream of it.
