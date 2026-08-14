# Engine registry

The record a step's `engine:` names, the invariants every row satisfies, and the rule that decides
which rows exist. This page is normative.

The registry is not part of the layered workspace and is never contributed by a layer. Membership in
it is the permission to dispatch, so a layer that could add its own rows could grant itself an
engine; see [ADR-0010](../decisions/ADR-0010-resolve-the-workspace-per-file.md).

## The engine record

An engine record carries exactly four fields and no others.

```yaml
- id: "<provider>-<model>-<effort>"
  provider: "<provider>"
  model: "<model>"
  effort: "<effort>"
```

| Field      | Type   | What it names                                             |
| ---------- | ------ | --------------------------------------------------------- |
| `id`       | string | the literal a step declares; derived from the other three |
| `provider` | string | which runner dispatches this row                          |
| `model`    | string | the provider's own name for the model                     |
| `effort`   | string | one rung of that provider's ladder                        |

The id is derived as `<provider>-<model>-<effort>`. It is written out rather than computed at read
time, so a row can be found by the literal an author typed without the reader reconstructing it.

## Provider metadata

Each provider declares its runner and its own ordered effort ladder.

```yaml
<provider>:
  runner: "<runner name>"
  efforts: ["<weakest>", "...", "<strongest>"]
```

Ordering inside a provider's ladder is meaningful. Ordering across providers is undefined: one
provider's middle rung is not comparable to another's, and no page in this specification claims
otherwise.

There is no field recording that a runner is planned rather than built. A provider whose runner does
not yet satisfy [runner contract](./runner-contract.md) has no rows, because a row that validates and
cannot be dispatched is the one thing membership-is-permission exists to rule out. Adding a provider
is therefore the same act as shipping its runner.

## The five invariants

Every row MUST satisfy all five:

1. Derived id: `id` equals `<provider>-<model>-<effort>`.
2. Unique id: no two rows share one.
3. Exactly four fields: an extra key is a typo, not an extension point.
4. Provider-valid effort: `effort` is a member of that provider's declared ladder.
5. Declared runner: the provider declares a runner name.

The fifth invariant is what the validator can check. That a declared runner exists and conforms is not
checkable from the registry, and it is the obligation membership carries rather than a field a row
asserts.

## Membership is the permission

An engine that cannot be run is a row that does not exist, not a row carrying a false flag.

Two consequences follow, and both show up as absences a reader would otherwise report as omissions. A
model whose provider rejects a particular rung never gets that row, because the row could never be
dispatched. And a rung that a provider accepts only by ignoring it — where passing the value silently
yields a different effort than the record names — is either omitted or dispatched by omitting the
flag, never forwarded verbatim.

## What ships

No rows. The registry's shape, its invariants, and its ladder rule are specified here; the rows
themselves are perishable vendor data an implementation supplies and revalidates.

Concrete providers, models, and effort rungs change without notice and without any local change, so
they are exactly the class of fact that belongs in a tracking registry with a cadence and a
`last_checked` date rather than in a specification. An implementation that ships rows MUST track them
that way.
