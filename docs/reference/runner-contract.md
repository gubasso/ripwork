# Runner contract

What a provider runner must do to be dispatchable. A runner is the component that turns one
fresh-context step into one running agent and one set of durable artifacts. This page is normative,
and it is deliberately written from the shape providers have in common rather than from any one of
them.

Its neighbour owns a different boundary: [orchestrator contract](./orchestrator-contract.md) owns
what a driver owes ripwork. This page owns the provider boundary — what every runner normalizes, and
what it deliberately leaves provider-shaped.

## What a runner is

A runner is a launcher, not a supervisor. It builds one argument vector, hands it to a durable job,
and returns. The job runs detached, so that killing the process tree the runner was called from
cannot reach it, and the outcome is reconstructed from files rather than from an observer that may
not survive.

A runner therefore owns exactly three things: the argument vector, the precondition report, and the
artifact paths. It owns no credential, no model policy, no scheduling, and no interpretation of what
the agent produced.

## Conformance fields

Ten fields. A runner is conformant when it supplies all ten and normalizes the ones marked as
normalized. A provider-owned field is named here so a reader knows it exists, not standardized.

| Field               | Conformance requires                                                     | Normalized |
| ------------------- | ------------------------------------------------------------------------ | ---------- |
| Launch argument vector | One vector the runner builds, never a string a caller can extend      | No         |
| Durable state document | One document at a caller-named absolute path                          | Yes        |
| Output artifact     | The agent's final message, as one file                                   | Yes        |
| Event stream        | The provider's own structured stream, captured whole                     | Yes        |
| Error stream capture | A separate file; never merged into the event stream                     | Yes        |
| Exit classification | `0` done and ok, `1` done and failed, `75` still running                 | Yes        |
| Effort selection    | One rung of the provider's own ladder, resolved from the engine record   | No         |
| Access posture      | Declared per launch and defaulting closed                                | No         |
| Account selection   | Named by the caller; the runner selects, never creates or rotates        | No         |
| Resume identity     | A durable handle naming the same conversation on a later call            | No         |

The five normalized fields are the ones a driver reads. The five provider-owned fields are where
providers genuinely differ, and normalizing them would mean inventing a shape none of them has.

## Durable artifacts

The durable state document carries every artifact path under one object: the output stream, the error
stream, the final message, the exit-code file, and the completion marker. A runner adds no schema of
its own; its provider-specific facts go in the fields that exist for exactly that.

Every artifact path MUST be absolute. A relative path resolves against the process's working
directory, and a detached job's working directory is not the caller's to assume.

Artifacts are written by the job, never by the runner after the fact. A runner does not copy, rename,
summarize, or truncate them, because a file the runner rewrote is a file whose contents no longer
prove what the agent did. The one exception is named under the asymmetries below.

## Classification

One verb classifies. It polls up to a bounded wall time, then decides from the exit-code file, the
completion marker, and the captured streams: `0` done and ok, `1` done and failed, `75` still
running.

A still-running job is never an error, and the driver owns the repetition — it re-issues the bounded
call while it sees `75`, which is the same retry shape the run verbs use.

The durable lifecycle states are `running`, `exited`, `finalized-ok`, `finalized-failed`, `cancelled`,
and `lost`. `lost` is reconstruction after a reboot or a vanished process group, not a timeout:
duration is never a failure signal.

Beyond that a runner MAY report a provider-shaped status of its own. Those statuses are diagnostics,
never a second classification. A caller branches on the exit code.

## Preconditions

A runner reports version and authentication preconditions and manages neither. It checks them before
the durable state document exists; once a job exists, its status belongs to the agent, so a
precondition that would have failed the launch must be caught while the runner is still the thing
answering. [ADR-0013](../decisions/ADR-0013-preflight-preconditions-before-the-durable-job.md) owns
the choice.

Reportable preconditions are the standing ones a retry reproduces:

- the provider's program is missing or not executable;
- no account is bound or selected;
- a required profile does not resolve;
- the underlying program is below a version floor the provider enforces;
- a credential is absent or refused.

Each is reported with what was checked and what would satisfy it, and none is fixed by ripwork.
Creating, storing, rotating, and selecting credentials are out of scope for every runner.

## The asymmetries the contract absorbs

Providers are not shaped alike, and the contract absorbs the difference rather than assuming it away.
Five classes recur, and a runner is written against the class rather than against one provider's
current spelling.

1. Some providers expose a dedicated non-interactive verb; others treat a headless run as a
   passthrough where every token after the launcher's own arguments reaches the child unchanged. The
   contract therefore requires a vector, not a verb.
2. Some providers require a bound account and a resolved profile before any launch; others have no
   such concept. Binding is named here as a precondition and its shape stays the provider's.
3. Some providers reject a ladder rung by warning and silently substituting a default. Such a rung
   MUST be dispatched by omitting the argument rather than forwarding the rung name, or the run
   silently executes at a different effort than the engine record names.
4. Some providers write the final message to a named file; others emit it inside the event stream. In
   the second case the runner extracts it and writes the output artifact itself. That is the one place
   a runner produces an artifact rather than letting the job write it, and it is why the output
   artifact is a conformance field rather than a provider detail.
5. Some launchers replace themselves with the child rather than supervising it, so they leave no
   post-flight and their exit range may overlap the agent's. Classification from artifacts alone then
   cannot separate never launched from ran and failed, which is what preflight exists to keep apart.

## What is not normalized

Provider diagnostics are preserved rather than translated. A runner does not map a provider's error
taxonomy into ripwork's, does not rewrite its error stream, and does not summarize its event stream.
Everything the provider said stays readable in the artifacts under its own vocabulary, and the
normalized fields above are what a caller branches on.

The reason is a standing cost. Providers keep diverging in lifecycle semantics, and a translation
layer over that divergence has to be corrected on every provider release. Normalizing five fields and
preserving the rest is the smaller obligation.
