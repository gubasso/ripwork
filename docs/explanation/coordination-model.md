# Coordination model

Why ripwork splits a workflow run across three parties, what each one owns, and what the split costs.
This page explains the forces; the contracts that bind each party are in
[reference](../reference/orchestrator-contract.md), and the choices behind them in
[decisions](../decisions/README.md).

Nothing here describes built software. No implementation exists, and this page will be replaced by
subsystem pages when one does.

## The problem the split solves

A multi-step agent workflow has three jobs in it. Something decides what runs next. Something runs
it. Something decides whether the result was good enough to move on.

A tool that takes all three becomes a client of every agent runtime it supports, and acquires new
work on every vendor release. It also has to hold an opinion about what a good result looks like,
which means reading what a probabilistic producer wrote and deciding it was adequate — a judgment no
schema can encode and no test can settle.

A tool that takes none of the three is a file format.

ripwork takes exactly the part that is decidable without reading an artifact or launching a process.

## What each party owns

```text
   definition                run                      agent
        │                     │                         │
   ┌────▼─────┐        ┌──────▼──────┐           ┌──────▼──────┐
   │ ripwork  │        │   driver    │           │   runner    │
   ├──────────┤        ├─────────────┤           ├─────────────┤
   │ validate │◄──────►│ dispatch    │──────────►│ launch      │
   │ readiness│  verbs │ judge       │  argv     │ preflight   │
   │ claims   │        │ recover     │           │ artifacts   │
   │ receipts │        │             │           │             │
   │ rounds   │        │             │           │             │
   └──────────┘        └─────────────┘           └─────────────┘
    counts files        reads them                writes them
    never reads         never writes state        never interprets
```

ripwork owns structure. Whether the graph is acyclic, whether a handle is unique, whether an engine
exists, whether a node's edges are satisfied, whether a token is live, whether a round produced a
file, and whether a ceiling has been reached. Every one of those is true or false without opening a
file.

The driver owns judgment and dispatch. Which node to run now, how to run it, whether a loop's prose
criterion has been met, and what to do when a person is needed. It is also the party that recovers: a
run survives its driver dying because the driver reconciles the durable record on restart, not
because anything supervised it.

The runner owns the provider boundary. One argument vector, one precondition report, one set of
artifact paths. No credential, no model policy, no scheduling, and no interpretation of what came
back.

## Why the driver is deliberately underspecified

The [orchestrator contract](../reference/orchestrator-contract.md) has a section listing what a driver
is not required to have: sub-agents, a depth budget, parallelism, streaming, a tool-call protocol, a
planning mode, an account.

That list is load-bearing rather than reassuring. It is the whole of what makes the contract
vendor-neutral, and it is why a shell script is a conformant driver. Any obligation added that a shell
script could not satisfy would quietly convert ripwork from a coordinator into an integration with
whichever runtime the new obligation assumed.

Capability negotiation exists so that the underspecification does not become guesswork. A driver
declares what it can do once, and ripwork enforces rather than trusts: a capability nobody declared is
a capability nobody exercises, so a limited driver gets a run it can honour instead of a silent
substitution it cannot.

## Why nothing is enforced about content

The producer of every artifact in a run is probabilistic. A contract it cannot honour is a contract
that fails at the most expensive possible moment — at depth, after real spend, on a run that was
otherwise going fine.

So the enforcement line sits at the directory. A step writes into its own directory, `needs:` hands
upstream directories to whoever depends on them, and a receipt inventories what landed. Declared
inputs and artifacts remain in a step file as description, aligned against the brief at lint time and
never checked at run time.

The same reasoning produces the prose stopping criterion. A loop's `until:` is a sentence a reader
applies to a round's directories; ripwork stores it and never parses it. What ripwork still holds is
everything countable: the ceiling, the token, the two closed vocabularies, and the rule that a round
producing no files can neither continue nor converge.

### The prose criterion is the majority position, not a shortcut

The split is not novel, and it tracks one variable. Surveyed across existing workflow frameworks, the
ones whose steps are executed deterministically state their stopping condition as a machine predicate,
and the ones whose steps are executed by a model state it in prose and bound it with a hard ceiling.
ripwork's executors are models, so prose follows. Choosing a predicate here would mean comparing
values a probabilistic producer was merely asked to write.

Two properties of that majority position are worth carrying, because both are load-bearing and neither
is obvious.

The ceiling is universal and always enforced by the framework, never left to the agent's cooperation.
Documented runaway loops in this class are loops whose ceiling was checked somewhere other than the
component that writes the transition, so an agent that declined to stop was never actually stopped.
That is why `max_rounds:` is checked inside `advance`, which is the only writer of a later round, and
why reaching it is a failure rather than a success.

Real ceilings are small. Published guidance for this shape clusters between four and ten rounds, which
is why the workspace default is a single digit: a ceiling large enough to hide a runaway loop only
makes the run expensive before it fails.

## What the split costs

- A driver is mandatory. ripwork alone runs nothing, so a user without one has a validator.
- A workflow file is a dependency graph and does not show what flows between steps. That is learned
  from the briefs, which makes step-to-brief alignment the only static check on data flow — so that
  check has to actually exist.
- A premature convergence is possible. It is auditable from the per-round record and is not
  prevented.
- Exclusion across a scope boundary is coarse, because the only edge available points at siblings.
- Ownership never expires, so a node whose owner vanished stays claimed until a driver reclaims it.

Each cost was accepted deliberately and is recorded where the choice was made rather than only here.

## Unresolved

- Whether a `failed` node has any path back. Nothing currently moves one, so the node and everything
  downstream of it are blocked, and no verb says what a user does about that.
- Whether coarse cross-scope exclusion needs a harder mechanism, and what real case would force one.
  The accepted answer is to wait for the case rather than build for the hypothesis.
- Whether three workspace layers earn themselves. The argument for them is reuse a user can shadow;
  the argument against is that a greenfield tool with one root has no drift to resolve yet.
- How a driver hands a brief to a runtime whose agent has no file-reading step. Every current shape
  assumes the agent can be pointed at a path.
