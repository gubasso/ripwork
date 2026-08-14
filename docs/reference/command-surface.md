# Command surface

Every verb, its arguments, the document it emits, and the exit protocol. This page is normative and
is the sole owner of the verb grammar; a page that needed a verb's spelling links here.

The command is `ripwork` and the verbs sit directly under it;
[ADR-0012](../decisions/ADR-0012-expose-one-command-with-top-level-verbs.md) owns why there is no
namespace between them.

## Output and the exit protocol

`--json` selects the machine-readable document, and that document is the contract. Without it a verb
prints a compact line for a person reading a terminal, and that line is a convenience with no
stability promise.

Every document carries a `schema` field keyed `ripwork.<verb>.v1`. A verb's shape may change only by
taking a new version, so a driver that checks the key cannot be handed a document it does not
understand.

Four exit codes, and no fifth. The code describes the call, never the run;
[ADR-0011](../decisions/ADR-0011-make-the-exit-protocol-protocol-only.md) owns the split.

| Code | Meaning |
| ---- | ------- |
| `0`  | a valid result, or an applied transition — including a correctly reported terminal `failed` |
| `1`  | an internal failure reading or persisting state |
| `2`  | invalid input: a stale or missing token, an illegal transition, a digest mismatch, or malformed arguments |
| `75` | cannot advance yet; nothing is wrong and the caller re-issues |

Every verb fails closed and reports `2` on invalid input. A driver that branches on the exit code
alone will read a terminal `failed` run as a healthy one, because reporting that state correctly is
itself a `0`. Only `done` is success, and it is read from the document.

## The verbs

```text
ripwork list    [--json]
ripwork show    <key> [--json]
ripwork init    <key> --from <key> [--force] [--json]
ripwork validate [<key>] [--all] [--json]
ripwork resolve --run-dir <dir> --key <key> --task-file <file>
                [--max-fresh-depth <n>] [--orchestrator <json>] --json
ripwork next    --run-dir <dir> --json
ripwork claim   --run-dir <dir> --as <handle> --owner <id>
                [--dispatch <text>] --json
ripwork record  --run-dir <dir> --as <handle> --claim-token <tok>
                --status done|failed [--reason <text>] --json
ripwork advance --run-dir <dir> --loop <handle> --decision-token <tok>
                --outcome continue|converged|pause|abort
                --reason criterion-met|stalled|unfeasible|needs-user
                [--note <text>] --json
ripwork reclaim --run-dir <dir> --as <handle> --previous-claim <tok>
                --owner <id> --reason <text> [--dispatch <text>] --json
ripwork summary --run-dir <dir> --json
ripwork conformance --run-dir <dir>
```

The first four read a workspace. The rest read or write one run directory.

### list

Reports every workflow definition the layers resolve, with the layer each one came from.

```text
ripwork.list.v1 — {schema, ok, action, workflows: [{key, layer, path}]}
```

### show

Reports one definition, its resolved step definitions, the workspace defaults in effect, and the
resolved source per file. The per-file source is what makes shadowing visible rather than inferred.

```text
ripwork.show.v1 — {schema, ok, action, key, files: [{role, key, layer, path}], meta, definition}
```

### init

Copies a definition into the project layer so it can be edited without forking the rest of the
workspace. It writes only paths it creates, and refuses to overwrite without `--force`.

```text
ripwork.init.v1 — {schema, ok, action, key, from, written: [path]}
```

### validate

Applies every rule in [workflow definition](./workflow-definition.md) and every invariant in
[engine registry](./engine-registry.md). With a key, one definition; with `--all`, the whole
workspace.

`findings` is a list rather than a first error, because a definition with four mistakes should
report four. `ok` is false when any finding is present, and the exit code is `2`.

```text
ripwork.validate.v1 — {schema, ok, action, checked: [key], findings: [{key, path, rule, detail}]}
```

The registry invariants run on every call rather than behind a separate mode. A breadth check that
has to be asked for is a gate that gets skipped.

### resolve

Validates first and materializes second, so an invalid definition leaves no directories behind. It
writes the run-state document, creates one directory per node, and records the driver's declaration.

`--task-file` names the task the run is for. `--orchestrator` carries the driver's declaration, and
`--max-fresh-depth` its opt-in depth ceiling; both are owned by
[orchestrator contract](./orchestrator-contract.md).

```text
ripwork.resolve.v1 — {schema, ok, action, run_dir, key, state,
                      nodes: [{as, id, dir, needs, engine}]}
```

`run_dir` and every path inside the document are absolute. A relative path resolves against a working
directory a dispatched agent's is not the caller's to assume.

### next

Reports the frontier: one node that is ready, or the run's state when none is.

A node is ready when it is pending and every `needs:` target has reached `done`. Absent the `parallel`
capability, `next` returns at most one node.

```text
ripwork.next.v1 — {schema, ok, action, state, reason, lease_age, requires_judgment,
                   node: {as, id, dir, engine, brief, inputs, dispatch} | null}
```

`inputs` names the upstream directories ripwork has chosen for this node. A driver MUST hand the node
those, rather than any path it derived itself.

When a claimed node blocks the frontier and the run is still live, `next` reports `75` with `node`
null. Nothing is wrong; the caller polls again. `reason` names the node holding the frontier and
`lease_age` reports how long it has been claimed. The lease age is information for a person to read
and is compared by nothing; ownership never expires.

`reason`, `lease_age`, and `requires_judgment` are present only in the cases that produce them.
`requires_judgment` is the directive described in [orchestrator contract](./orchestrator-contract.md),
which owns what a driver must do with it.

### claim

Takes ownership of one node and mints a claim token. The token is the only thing `record` accepts.

`--dispatch` is optional and carries whatever the driver needs to find this work again after a
restart. ripwork stores it, returns it on `next` and in the receipt, and never parses it. A driver
that supplies nothing gets a null back and owns the consequence, which is that a claimed node with no
receipt tells it nothing about whether the work ran.

```text
ripwork.claim.v1 — {schema, ok, action, as, owner, claim_token, dir, inputs, dispatch}
```

### record

Verifies the token, inventories the node directory, writes the receipt, and updates run state. One
call; there is no separate finish. The receipt carries the `dispatch` value from the claim.

A failed node still gets a receipt, carrying an error object. The receipt sees the node directory only,
never walks above it, and never opens a file inside it — the inventory is names and sizes.

```text
ripwork.record.v1 — {schema, ok, action, as, status, receipt_path, outputs_count, run_state}
```

The receipt's own shape and its closed key set are [run directory](./run-directory.md).

### advance

The only writer of a round after the first. A driver reports its judgment of a loop's criterion here,
and ripwork applies or refuses the transition.

Round 1 is not written here. It is materialized when the loop node itself becomes ready, because a
loop that produced nothing cannot report an outcome and `continue` is refused on an empty round —
which would leave a loop that never starts. Entering a loop creates its first round; every round after
that is a decision.

`--outcome` and `--reason` are both closed vocabularies:

| Outcome     | What it asserts                          |
| ----------- | ---------------------------------------- |
| `continue`  | the criterion is unmet; materialize a round |
| `converged` | the criterion is met; the loop is done   |
| `pause`     | stop here, resumable                     |
| `abort`     | stop here, terminal failure              |

| Reason          | When it applies                                |
| --------------- | ---------------------------------------------- |
| `criterion-met` | the authored criterion was satisfied           |
| `stalled`       | rounds are no longer changing the outcome      |
| `unfeasible`    | the driver judges the run cannot succeed       |
| `needs-user`    | the decision requires a person                 |

Three transitions are illegal and report `2`: advancing past `max_rounds`, and reporting `continue`
or `converged` on a round that produced no files. `pause` and `abort` stay legal on an incomplete
round, so an empty round can still be escaped.

There is no `--round` argument. ripwork owns the counter, and the decision token already binds the
round.

```text
ripwork.advance.v1 — {schema, ok, action, loop, outcome, reason, note, round, state,
                      requires_judgment}
```

`requires_judgment` is present only when the run cannot proceed without a person, and
[orchestrator contract](./orchestrator-contract.md) owns what a driver must do with it.

An applied transition is durable and its decision token is spent, so replaying the same token MUST NOT
be accepted a second time.

### reclaim

Takes a node whose previous owner did not return. It requires the previous claim token and a reason,
and mints a new claim token.

```text
ripwork.reclaim.v1 — {schema, ok, action, as, owner, claim_token, reason}
```

Ownership never expires. No elapsed time reclaims a node; a driver does, and the reason it gives is
retained.

### summary

Reports the run and every node's status and output count. It reports and never gates.

```text
ripwork.summary.v1 — {schema, ok, action, run_dir, key, state,
                      nodes: [{as, status, outputs_count, engine}]}
```

### conformance

The readout a person or a continuous-integration job reads to see whether a driver honoured the
contract. It is the one verb with no machine-readable document, because its audience is a reader
deciding whether to trust a driver, not a driver deciding what to do next.

## Run states

Terminal states live in the document's `state` field, never in an exit code.

| State     | Meaning                                              |
| --------- | ---------------------------------------------------- |
| `running` | live; work remains or is claimed                     |
| `paused`  | stopped by a driver's report, resumable              |
| `done`    | terminal; every node reached `done`                  |
| `failed`  | terminal; a node failed or a loop aborted            |

`paused` is a report from the driver rather than a function of the node statuses, which is why it is
the one state that is not recomputed.
