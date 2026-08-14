# Open questions

A question here blocks named entries until it is answered. A question that blocks nothing belongs in
the drafts workspace.

`Blocks:` is a comma-separated id list before an optional em dash and its reason. Every question exits
through a decision record, a story revision, or a recorded measurement.

## Q-001 — Do three workspace layers earn themselves in a greenfield tool?

Blocks: 002 — the story delivers whichever answer this question gives.

The argument for layers is reuse a user can shadow: a project-layer workflow references a shipped
step, and overriding one file costs one file. The argument against is that a tool with one root has no
drift to resolve yet, and every layer is a place a reader has to look before knowing what will run.
[ADR-0010](../decisions/ADR-0010-resolve-the-workspace-per-file.md) chose per-file resolution across
three layers, but it chose the resolution rule rather than the layer count — a single-root
implementation obeys every sentence of that record.

Exit: an amendment to ADR-0010 fixing the layer count, or a recorded decision that the count is an
implementation choice the specification does not fix.

## Q-002 — Does a failed node have any path back?

Blocks: 010, 016 — the receipt writes the terminal status, and recovery is where a user would expect
the way out to be.

Nothing currently moves a node out of `failed`. Such a node is blocked and so is everything downstream
of it, which makes one bad step the end of a run that may be nearly complete. The alternatives are all
real and all cost something: a retry verb reintroduces the scheduling ripwork refuses; a reclaim that
also resets the status weakens the single-writer story; and leaving it terminal is defensible only if
resolving a fresh run from the same definition is cheap.

Exit: a decision record naming the path, or one naming its deliberate absence and the workflow a user
follows instead.

## Q-003 — Does coarse cross-scope exclusion need a harder mechanism?

Blocks: 013 — flattening composites into sibling handles is where the coarseness becomes visible.

[ADR-0006](../decisions/ADR-0006-express-exclusion-with-needs.md) accepted that two conflicting leaves
in different composites can only be separated by serializing the composites that contain them, and
accepted it on the grounds that a harder check should wait for a real case rather than be built for a
hypothesis. The question is what a real case looks like, so the wait has an end condition rather than
being indefinite by default.

Exit: a recorded case that the edge cannot express, and a decision record on it; or a recorded
decision that the accepted cost stands and this question closes.

## Q-004 — How does a driver hand a brief to an agent that cannot read a file?

Blocks: 021 — the conformance fields assume an argument vector and a set of paths.

Every current shape assumes the agent can be pointed at a path: the brief is a file, the upstream
inputs are directories, and the runner passes locations rather than content. A provider whose agent
takes only an inlined prompt breaks that assumption, and the two repairs are not equivalent — a runner
that inlines the brief is reading a file it was told never to interpret, while a driver that inlines
it takes on a job the contract gave the runner.

Exit: a decision record placing the responsibility, or a measurement showing every provider worth
supporting can be pointed at a path.
