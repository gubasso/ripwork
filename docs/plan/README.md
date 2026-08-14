# Plan

What ripwork is building next, and what bounds it.

## Where to start

The topmost entry of `lanes/doing.yml` is the work in flight. When that lane is empty, the topmost
entry of `lanes/todo.yml` is what to start: ranking is legal by construction, so the first entry is
always eligible. Open its story under `stories/`, and load the individual sources its `Governed by`
section names.

## What bounds it

[charter.md](./charter.md) states the outcome, the pillars every story is judged against, and the
no-gos. `.plan-xp.yml` at the repository root carries the plan directory and the iteration cadence a
tool reads. An entry may name an epic under `epics/`, which holds an end state no single story
delivers.

## What can stop it

`needs` on an entry names the ids that must close first. [open-questions.md](./open-questions.md)
names decisions that block specific ids until they are answered. Neither is a stored status; both are
read from the record.

## What already happened

`lanes/closed.yml` is an append-only log ordered by close date. It is history, not a queue. It is
empty: nothing has been built.

## The gate

The method is [AGENTS.md](./AGENTS.md), which is self-sufficient in this repository. The linter that
enforces it is not: it comes from an installed plan-xp, and this repository deliberately vendors
neither it nor its schemas, because a copy here would drift from the version a gate actually runs.
Until one is installed, the lane files and the ranking rules are held by review.
