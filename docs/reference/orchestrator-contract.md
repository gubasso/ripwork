# Orchestrator contract

What a driver of ripwork must be, must do, and must not do. An orchestrator here is any program that
drives a run to a terminal state — a coding agent of any kind, a continuous-integration job, or a
shell script. This page is the conformance target, and it is normative.

Two neighbours own what this page deliberately does not.
[Workflow definition](./workflow-definition.md) owns the grammar and the validator rules.
[Command surface](./command-surface.md) owns the verb grammar and the exit protocol.

## Declaration

A driver declares itself once, at `resolve`, through `--orchestrator`:

```json
{"name": "<driver name>", "version": "1", "capabilities": ["inline", "subprocess", "ask-user"]}
```

`name` and `version` identify the driver in run state and in a conformance readout. `capabilities` is
a set drawn from the closed vocabulary below; an unknown member is invalid input.

ripwork enforces rather than trusts. A capability the driver did not declare is a capability ripwork
refuses to exercise, so a driver that under-declares gets a run it can honour, and a driver that
over-declares fails at the first directive it cannot honour rather than silently degrading.
Declaration is not a hint to be second-guessed at dispatch time.

## The capability vocabulary

Four members. Each row states the consequence of the capability being absent, because absence is the
safe default and the thing a conformant driver must survive.

| Capability   | Absent means                                            |
| ------------ | -------------------------------------------------------- |
| `parallel`   | `next` returns at most one dispatch directive            |
| `inline`     | a node declaring `context: inherit` fails closed         |
| `ask-user`   | a `requires_judgment` directive becomes a hard failure   |
| `subprocess` | only nodes declaring `context: inherit` are dispatchable |

`inline` and `subprocess` range over the `context:` key on a node, owned by
[workflow definition](./workflow-definition.md#execution-context) and decided by
[ADR-0008](../decisions/ADR-0008-select-an-execution-context-at-its-node.md). They are independent. A
driver that can only fork declares `subprocess` alone and gets a run in which every `inherit` node
fails closed rather than being quietly forked. A driver that can only run in session declares `inline`
alone. A driver declaring both may be handed a definition mixing the two.

`requires_judgment` is a directive ripwork emits, not a field a driver invents. A `next` or `advance`
document MAY carry it, with a reason, when the run cannot proceed without a person. The accepted
grammar produces exactly one such case: a loop's `until:` criterion, which
[ADR-0007](../decisions/ADR-0007-judge-loop-convergence-with-a-prose-criterion.md) makes a prose
judgment ripwork stores and never parses, reaching `advance` as the `needs-user` reason already in its
closed set. A driver holding `ask-user` surfaces the directive to a person; a driver without it stops.

## Driver obligations

A conformant driver MUST:

- run a command and read its output and its exit code;
- parse the machine-readable document;
- declare itself once, at `resolve`;
- drive `next`, `claim`, dispatch, `record` while the run is live, re-issuing on `75`;
- dispatch by any means it has — a subprocess, a network call, another agent, or its own context;
- pass the claim token on `record` and the decision token on `advance`;
- hand each node the upstream directories ripwork names for it, rather than any path it derived
  itself;
- read a loop's `until:` criterion and the round's directories, then report its own judgment on
  `advance`;
- reconcile every durable `claimed` node after a restart, before selecting new work — an absent
  receipt does not say whether the work ran, so a driver reads back the `dispatch` value it supplied
  and asks its own runtime before choosing between recording an outcome and reclaiming;
- surface a `requires_judgment` directive to a person, or abort;
- adapt at run time and keep the run going on best effort, because nothing about what a step produced
  is enforced and a driver that halts on the first surprise halts on most runs;
- stop a run it judges unfeasible — a dead end, or a loop that is repeating without converging —
  rather than continuing to spend against it. Inside a loop that is an `abort` with the `unfeasible`
  reason; outside one it is recording the current node `failed`, which makes the run terminal;
- stop on a terminal state, and report at the end what it had to work around — a brief that did not
  produce what a downstream step needed, a step whose declared artifacts no longer match it, an edge
  that turned out to be missing. ripwork enforces none of that at run time, so a driver adapting
  silently is the only thing standing between a gap in a definition and a person who could close it.

Two of those obligations pull against each other on purpose. Adapting keeps a run alive through a
surprise; judging it unfeasible ends one that cannot succeed. Which applies is the judgment ripwork
declines to make, and the reason vocabulary is where a driver records which it chose.

## Driver prohibitions

A conformant driver MUST NOT:

- edit the run-state document or any resolved definition;
- write any receipt — `record` is the only writer;
- synthesize an id, a handle, a token, or a counter;
- infer a round count rather than reading the one ripwork reports;
- re-dispatch a claimed node without `reclaim`;
- treat any terminal state but `done` as success.

The prohibitions are what make single-writer discipline hold. A driver that edits state directly is
not a slower conformant driver; it is a driver whose run cannot be recovered after a crash, because
the durable record no longer describes what happened.

## What is explicitly not required

Sub-agents. A depth budget. Parallelism. Streaming. A tool-call protocol. A planning mode. A model, a
provider, or an account. Any particular runtime, language, or transport.

This list is load-bearing rather than reassuring. It is the whole of what makes the contract
vendor-neutral, and it is why a shell script that calls `ripwork` and dispatches with an ordinary
subprocess is a conformant driver. Anything added to the obligations above that such a script cannot
satisfy is a contract regression.

## Depth

`--max-fresh-depth` is an opt-in contract, not a base number ripwork supplies.

Absent, ripwork enforces no depth ceiling and the driver owns whatever limit its own runtime has.
Present at `resolve`, it is the driver's declared ceiling, and ripwork refuses a dispatch that would
exceed it rather than letting the failure land at depth after real spend.

A runtime's own nesting limit is a fact about that runtime. It is not this contract's number and MUST
NOT be written into this page: a driver with no nesting has no such ceiling, and a driver with a
different one declares it.

## Terminal states

Terminal states live in the document's `state` field, never in the exit code. The exit protocol is
protocol only, and [command surface](./command-surface.md) owns it.

Only `done` is success. A driver that branches on the exit code alone will read a terminal `failed`
run as a healthy one, because reporting that terminal state correctly is itself a `0`.
