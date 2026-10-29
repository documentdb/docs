---
title: collMod
description: Modify collection, view, schema-validation, and index settings in DocumentDB.
type: commands
category: administration
---

# collMod

Modifies supported collection or index properties.

The examples use the `route_no_1` index created by [`createIndexes`](../indexing/createIndexes.md) on the [airline dataset](../diagnostic/wire-protocol.md#airline-example-dataset).

## Syntax

```javascript
db.flights.hideIndex("route_no_1")
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`collMod`** | (Required) A string specifying the collection or view name. |
| **`index`** | (Optional) A document specifying the index selector and modification, described below. |
| **`viewOn`** | (Optional) A string that replaces a view's source. |
| **`pipeline`** | (Optional) An array that replaces the view pipeline; must be supplied with `viewOn`, not mixed with other modification categories. |
| **`validator`** | (Optional) A document that replaces schema-validation rules when schema validation is enabled. |
| **`validationLevel`** | (Optional) A string selecting `off`, `strict`, or `moderate`; rejected when schema validation is disabled. |
| **`validationAction`** | (Optional) A string selecting `error` or `warn`; rejected when schema validation is disabled. |
| **`enableStats`** | (Optional) A boolean provided as a DocumentDB extension controlling planner statistics. |
| **`colocation`** | (Optional) A document provided as a DocumentDB extension passed to the deployment's collection-colocation hook. The default single-node backend makes no change; nested options are deployment-specific. |
| **`changeStreamPreAndPostImages`** | (Optional) A document with the following behavior: `{ enabled: <boolean> }`; applied when pre-image support is enabled and cluster version is at least 0.112. Otherwise ignored. |
| **`enableUpdateDescription`** | (Optional) A boolean-like value that enables or disables change-stream update descriptions. |

The `index` document accepts:

| Parameter | Description |
| --- | --- |
| **`name`** | (Optional) A string that selects an index by name; provide this or `keyPattern`. |
| **`keyPattern`** | (Optional) A document that selects an index by its key specification. |
| **`newName`** | (Optional) A string that renames the selected index. |
| **`hidden`** | (Optional) A boolean-like value that changes index visibility, subject to index restrictions. |
| **`prepareUnique`** | (Optional) A boolean-like value that prepares a supported non-unique index for uniqueness conversion. |
| **`unique`** | (Optional) A boolean-like value that requests uniqueness conversion, subject to validation of existing data and index type. |
| **`expireAfterSeconds`** | (Optional) A non-negative number that changes the expiration interval for a TTL index. |
| **`reindex`** | (Optional) A boolean-like value that must be `true`. Enqueues the reindex request. Monitor progress via [`currentOp`](../diagnostic/currentOp.md). |
| **`updateOptions`** | (Optional) A boolean-like value that controls refreshing index options when rebuilding. |

Use `collMod.index.reindex` to rebuild an index. The DocumentDB-specific option has no dedicated shell helper:

```javascript
db.runCommand({ collMod: "flights", index: { name: "route_no_1", reindex: true } })
```

`dryRun` is not a supported top-level field. Schema-validation settings, including `validator`, are rejected if schema validation is disabled. Shared fields such as `writeConcern` and `comment` are accepted without changing the modification semantics. See [wire protocol and common fields](../diagnostic/wire-protocol.md).

See [Build a unique index on existing data](../indexing/createIndexes.md#build-a-unique-index-on-existing-data) for the staged `buildAsUnique`, `prepareUnique`, and `unique` workflow.

## Implementation

Source: `pg_documentdb/src/commands/coll_mod.c`, `ParseSpecSetCollModOptions` and `ParseIndexSpecSetCollModOptions`.
