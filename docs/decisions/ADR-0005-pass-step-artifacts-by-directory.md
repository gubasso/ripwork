# ADR-0005: Pass step artifacts by directory

## Context and Problem Statement

A step has to hand what it produced to the steps that depend on it. Declaring those artifacts as
typed handles, and failing a step whose declared artifact was missing or mistyped, makes ripwork a
runtime judge of content a probabilistic agent produced. It buys static checkability the producer
cannot honour, and it does so at the point in a run where failure is most expensive.

## Considered Options

- Declared typed handles, enforced at run time
- Declared typed handles, checked only at lint time
- Whole step directories passed along `needs:` edges

## Decision Outcome

Chosen option: `Whole step directories passed along needs: edges`.

Each step writes its artifacts into its own directory under the run directory, named by its `as:`
handle. `needs:` hands every upstream directory to the step that depends on it. A step file MAY
describe the inputs it expects and the artifacts it produces; that description is aligned against its
brief at lint time and is never enforced at run time. The brief owns what the step reads and
produces.

Deleted with the handles: the expression dialect, the reserved output filenames, the implicit
standard-output handle, the typed-scalar file, and the output-name collision question — directories
are namespaced by `as:`.

A receipt therefore inventories the node directory rather than verifying it against a declaration.
ripwork counts files and never reads them.

## Consequences

- Good: nothing in the run can fail on a mismatch between what an agent was told to write and what it
  wrote.
- Good: a step's output surface can grow without any caller changing.
- Bad: a workflow file is a dependency graph and no longer shows what flows between steps. A reader
  learns that from the briefs.
- Bad: step-to-brief alignment becomes the only static check on data flow, so that check has to
  actually exist rather than being assumed.

## Status

Accepted
