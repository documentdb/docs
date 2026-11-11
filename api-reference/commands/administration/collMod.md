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
| **`enableStats`** | (Optional) A boolean provided as a DocumentDB extension enabling or disabling per-collection planner statistics. Requires per-collection planner-statistics support; cannot be used on a view. |
| **`colocation`** | (Optional) A document provided as a DocumentDB extension passed to the deployment's collection-colocation hook. The default single-node backend makes no change; nested options are deployment-specific. |
| **`changeStreamPreAndPostImages`** | (Optional) A document with the following behavior: `{ enabled: <boolean> }`; applied when pre-image support is enabled and cluster version is at least 0.112. Otherwise ignored. |
| **`enableUpdateDescription`** | (Optional) A boolean-like value that enables or disables change-stream update descriptions. |

For additional behaviors, see the [contrib section](../contrib/).

The `index` document accepts:

| Parameter | Description |
| --- | --- |
| **`name`** | (Optional) A string that selects an index by name; provide this or `keyPattern`. Renaming requires both. |
| **`keyPattern`** | (Optional) A document that selects an index by its key specification. Renaming requires both this and `name`; other index modifications accept only one selector. |
| **`newName`** | (Optional) A string provided as a DocumentDB extension that renames the selected index. Requires both `name` and `keyPattern` and cannot be combined with other modifications. |
| **`hidden`** | (Optional) A boolean-like value that hides an index from query planning when `true` or makes it visible when `false`. Cannot modify visibility of an invalid, unique, or `_id_` index. |
| **`prepareUnique`** | (Optional) A boolean-like value that prepares a supported non-unique index for uniqueness conversion. |
| **`unique`** | (Optional) A boolean-like value that requests uniqueness conversion, subject to validation of existing data and index type. |
| **`expireAfterSeconds`** | (Optional) A non-negative number that changes the expiration interval for a TTL index or converts an eligible index to TTL. |
| **`reindex`** | (Optional) A boolean-like value that must be `true`. Enqueues the reindex request. Monitor progress via [`currentOp`](../diagnostic/currentOp.md). |
| **`updateOptions`** | (Optional) A boolean-like value that controls refreshing index options when rebuilding. Can only be supplied with `reindex`; cannot be enabled for the `_id_` index. |

`dryRun` is not a supported top-level field. Schema-validation settings, including `validator`, are rejected if schema validation is disabled. Shared fields such as `writeConcern` and `comment` are accepted without changing the modification semantics. See [wire protocol and common fields](../diagnostic/wire-protocol.md).

See [Build a unique index on existing data](../indexing/createIndexes.md#build-a-unique-index-on-existing-data) for the staged `buildAsUnique`, `prepareUnique`, and `unique` workflow.

## Examples

The following examples use the `bookings` database. Run the flight-index examples in order: the visibility example restores the index before reindexing, and the rename example changes its name for subsequent requests.

### Enable planner statistics

Before enabling statistics, inspect the [`listCollections`](listCollections.md) output for `flights` (for example, with `db.getCollectionInfos({ name: "flights" })`). If `options.statsEnabled` is `true`, planner statistics are already enabled.

Enable per-collection planner statistics for `flights`:

```javascript
db.runCommand({ collMod: "flights", enableStats: true })
```

The planner can use statistics about indexed values to estimate query selectivity when choosing a plan. This option requires the deployment's per-collection planner-statistics capability and applies to collections, not views. It is not a collection-size report like [`collStats`](../diagnostic/collStats.md), and enabling it does not guarantee a particular query plan.

To disable these planner statistics for the collection:

```javascript
db.runCommand({ collMod: "flights", enableStats: false })
```

Disabling removes the associated planner statistics, not the indexes or collection data. To refresh statistics while keeping them enabled, see [`compact` with `mode: "updateStats"`](../diagnostic/compact.md).

### Hide and unhide an index

Temporarily exclude `route_no_1` from query planning without dropping it, then restore its visibility:

```javascript
db.flights.hideIndex("route_no_1")
db.flights.unhideIndex("route_no_1")
```

These shell helpers set `index.hidden` through `collMod`. Hiding keeps the index and its definition, so unhiding does not require creating a new index. Visibility changes require a valid, non-unique index other than `_id_`.

### Rebuild an index

Use `collMod.index.reindex` to rebuild an index. The DocumentDB-specific option has no dedicated shell helper:

```javascript
db.runCommand({ collMod: "flights", index: { name: "route_no_1", reindex: true } })
```

The request enqueues an asynchronous rebuild; a successful command response does not mean the rebuild has finished. Monitor the build using [`currentOp`](../diagnostic/currentOp.md#monitor-index-builds).

The index must be valid and visible, and cannot have `buildAsUnique` or `prepareUnique` enabled. Submit reindexing separately from other modifications. Its optional `updateOptions` setting can only accompany `reindex`.

### Rename an index

Give the route index a more descriptive name:

```javascript
db.runCommand({
  collMod: "flights",
  index: {
    name: "route_no_1",
    keyPattern: { route_no: 1 },
    newName: "flight_route_lookup"
  }
})
```

This DocumentDB extension requires both the existing name and its matching key pattern. It changes the name in index metadata without rebuilding the index or changing its keys. Afterward, use `flight_route_lookup` in name-based index operations.

The index must be valid, cannot be `_id_`, and its new name must not conflict with another index. Send the rename separately from other modifications.

### Update a TTL expiration interval

Use the application-managed `flight_events` collection from the [TTL example](../indexing/createIndexes.md#unique-blocking-and-ttl-indexes), rather than expiring the shared flight dataset. Create its date-field index with a 24-hour expiration interval, then shorten that interval to one hour:

```javascript
db.flight_events.createIndex(
  { recorded_at: 1 },
  { name: "recorded_at_ttl", expireAfterSeconds: 86400 }
)

db.runCommand({
  collMod: "flight_events",
  index: { name: "recorded_at_ttl", expireAfterSeconds: 3600 }
})
```

The new interval is measured from each event's BSON date in `recorded_at`, not from the time of the modification. Shortening it can make existing events immediately eligible for expiration, but deletion happens asynchronously.

`expireAfterSeconds` can also convert an eligible single-field, non-`_id` index into a TTL index; the same TTL index-type restrictions apply as at creation. Updating an existing TTL interval changes its metadata without rebuilding the index.

## Implementation

Source: `pg_documentdb/src/commands/coll_mod.c`, `ParseSpecSetCollModOptions`, `ParseIndexSpecSetCollModOptions`, and `ModifyIndexSpecsInCollection`; planner statistics: `pg_documentdb/src/metadata/index.c`, `UpdateCollectionPlannerStatistics`.
