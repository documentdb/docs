---
title: createIndexes
description: Create indexes with supported standard and DocumentDB-specific options.
type: commands
category: indexing
---

# createIndexes

Creates one or more indexes.

The examples use the [airline dataset](../diagnostic/wire-protocol.md#airline-example-dataset). The vector example indexes each flight's three-element `route_vector` and requires HNSW support.

```javascript
db.flights.createIndex({ route_no: 1 })
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`createIndexes`** | (Required) A string specifying the collection name. |
| **`indexes`** | (Required) An array of documents specifying the index specifications. |
| **`ignoreUnknownIndexOptions`** | (Optional) A boolean-like value that ignores unknown index-specification options instead of rejecting them; defaults to `false`. Does not allow arbitrary command-level fields. |
| **`blocking`** | (Optional) A boolean-like value provided as a DocumentDB extension requesting blocking index creation; defaults to `false`. |
| **`skipWaitForIndex`** | (Optional) A boolean-like value provided as a DocumentDB extension for submitting background builds without the usual wait, where supported by the build path. |
| **`maxTimeMS`** | (Optional) A number that limits gateway waiting for a background index build. A timeout does not imply the submitted build was cancelled. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

## Index specification

| Parameter | Description |
| --- | --- |
| **`key`** | (Required) A document specifying the indexed paths and index kinds, such as `1`, `-1`, `"text"`, `"2d"`, `"2dsphere"`, or `"cosmosSearch"`. |
| **`name`** | (Required on the wire) A string specifying the index name. Optional when using the shell's `createIndex()` helper. |
| **`unique`** | (Optional) A boolean-like value that enforces unique indexed values. |
| **`partialFilterExpression`** | (Optional) A document that indexes only documents matching this filter. |
| **`sparse`** | (Optional) A boolean-like value specifying the sparse-index behavior, subject to supported index combinations. |
| **`wildcardProjection`** | (Optional) A document specifying the included or excluded paths for a wildcard index. |
| **`v`** | (Optional) An integer specifying the index specification version; unsupported versions are rejected. |
| **`expireAfterSeconds`** | (Optional) A non-negative number specifying the TTL interval within the supported 32-bit range. |
| **`default_language`** | (Optional) A string specifying the default text-index language. |
| **`language_override`** | (Optional) A string specifying the document field selecting the text-index language. |
| **`weights`** | (Optional) A document or `"$**"` specifying the text-index field weights or wildcard weights. |
| **`textIndexVersion`** | (Optional) A number specifying the text-index version; validated when a new index must actually be built. |
| **`cosmosSearchOptions`** | (Optional) A document specifying the vector index options, described below. |
| **`min`** | (Optional) A number specifying the bounds for a `2d` index. |
| **`max`** | (Optional) A number specifying the bounds for a `2d` index. |
| **`bits`** | (Optional) An integer specifying `2d` precision, from `1` through `32`. |
| **`collation`** | (Optional) A document specifying the non-empty comparison rules; requires collation support. |
| **`prepareUnique`** | (Optional) A boolean that is recognized only by the prepare-unique creation path; not a generally available creation option. |
| **`storageEngine`** | (Optional) A document containing the DocumentDB-specific options below, not a choice of PostgreSQL storage engine. |

### DocumentDB-specific index options

These are accepted directly in an index specification or inside `storageEngine`:

| Parameter | Description |
| --- | --- |
| **`buildAsUnique`** | (Optional) A boolean-like value that builds an index for the [staged unique-index conversion workflow](#build-a-unique-index-on-existing-data), without declaring it unique. |
| **`blocking`** | (Optional) A boolean-like value requesting a blocking build for this index. `storageEngine.blocking` mirrors the top-level command's `blocking` option. |

### Regular and composite indexes

Both examples use the default ordered/composite index format without explicitly selecting it.

**Regular index:** Create a single-field index for arrival-airport lookups:

```javascript
db.flights.createIndex(
  { arrival_airport: 1 },
  { name: "arrival_airport_1" }
)

db.flights.find({ arrival_airport: "LED" })
```

**Composite (ordered) index:** Index the departure airport followed by scheduled departure time, for queries that filter by airport and sort by departure time:

```javascript
db.flights.createIndex(
  { departure_airport: 1, scheduled_departure: 1 },
  { name: "departure_airport_1_scheduled_departure_1" }
)

db.flights.find({ departure_airport: "SVO" }).sort({ scheduled_departure: 1 })
```

An ordered/composite index can also have a single field. The key document specifies the indexed fields and their order.

### Unique, blocking, and TTL indexes

**Unique index:** Enforce uniqueness of the combination of route number and scheduled departure. Existing documents must already satisfy the constraint.

```javascript
db.flights.createIndex(
  { route_no: 1, scheduled_departure: 1 },
  { name: "route_no_1_scheduled_departure_1_unique", unique: true }
)
```

**Blocking index:** Request a blocking build for this index through `storageEngine.blocking`. A blocking build can block writes to the collection while it runs.

```javascript
db.flights.createIndex(
  { departure_airport: 1 },
  {
    name: "departure_airport_1_blocking",
    storageEngine: { blocking: true }
  }
)
```

**TTL index:** An application can keep short-lived flight events in a separate `flight_events` collection. Store `recorded_at` as a BSON date and expire events 24 hours after that timestamp:

```javascript
db.flight_events.insertOne({
  flight_id: 101,
  event: "check_in_opened",
  recorded_at: new Date()
})

db.flight_events.createIndex(
  { recorded_at: 1 },
  { name: "recorded_at_ttl", expireAfterSeconds: 86400 }
)
```

TTL indexes must be single-field indexes and cannot target `_id`. Deletion happens asynchronously through TTL cleanup, not exactly at the expiration timestamp.

### Vector index options

| Parameter | Description |
| --- | --- |
| **`kind`** | (Required) A string specifying the index kind. Built-in kinds are `"vector-ivf"` and `"vector-hnsw"`; HNSW requires its server capability to be enabled. |
| **`similarity`** | (Required) A string selecting `"COS"`, `"L2"`, or `"IP"`. |
| **`dimensions`** | (Required) A number specifying the vector dimension count, greater than `1`, within the implementation's dimension limits. |
| **`compression`** | (Optional) A string selecting the compression mode. `"half"` requires half-vector support and its feature gate. |
| **`numLists`** | (Optional) A number specifying the IVF list count, within the implementation's supported range. |
| **`m`** | (Optional) A number specifying the HNSW graph connectivity, within its supported range. |
| **`efConstruction`** | (Optional) A number specifying the HNSW construction search width, within its supported range. |

Other vector implementations can register additional kinds and options; those are not part of the default built-in wire surface.

```javascript
db.flights.createIndex(
  { route_vector: "cosmosSearch" },
  {
    name: "route_vector_index",
    cosmosSearchOptions: {
      kind: "vector-hnsw",
      similarity: "COS",
      dimensions: 3,
      m: 16,
      efConstruction: 64
    }
  }
)
```

### Text index representations

Text indexes support two representations. The following examples describe the same text index over the flight's departure and arrival airport fields; choose one representation rather than creating two indexes.

**Field-name representation:** Set each indexed field to `"text"` in the key document. Fields have weight `1` unless overridden with `weights`.

```javascript
db.flights.createIndex(
  { departure_airport: "text", arrival_airport: "text" },
  { name: "flight_airports_text" }
)
```

**Metadata representation:** Use `{ _fts: "text", _ftsx: 1 }` as the key document and identify the actual indexed fields and their weights in `weights`. `_fts` and `_ftsx` are index metadata markers, not fields you need to add to your documents. This is also the key representation used in [`listIndexes`](listIndexes.md) output.

```javascript
db.flights.createIndex(
  { _fts: "text", _ftsx: 1 },
  {
    name: "flight_airports_text",
    weights: { departure_airport: 1, arrival_airport: 1 }
  }
)
```

For a wildcard text index, use `{ "$**": "text" }` in the field-name representation, or the metadata key document with `weights: { "$**": 1 }`.

## Build a unique index on existing data

Use this staged workflow to add uniqueness to a collection that already contains documents. It requires prepare-unique and unique-conversion support to be enabled in the deployment. The example is an alternative to the direct `unique: true` example above; start without that index already created.

### 1. Build the index for uniqueness conversion

Create the index with `storageEngine.buildAsUnique: true`, without setting `unique: true`. Wait for index creation to complete before proceeding. An ordinary non-unique index created without `buildAsUnique` is not eligible for this workflow.

```javascript
db.flights.createIndex(
  { route_no: 1, scheduled_departure: 1 },
  {
    name: "route_no_1_scheduled_departure_1_unique",
    storageEngine: { buildAsUnique: true }
  }
)
```

### 2. Prevent new duplicate keys

Use [`collMod`](../administration/collMod.md) to set `prepareUnique: true`. This starts rejecting writes that would introduce duplicate index keys; it does not resolve existing duplicates. Correct any existing duplicate combinations of route number and scheduled departure before the next step.

```javascript
db.runCommand({
  collMod: "flights",
  index: {
    name: "route_no_1_scheduled_departure_1_unique",
    prepareUnique: true
  }
})
```

### 3. Validate existing data and convert

Set `unique: true` in a separate `collMod` request. This validates existing indexed data and, if validation succeeds, converts the index to a unique index. If duplicate keys remain, conversion fails; correct the conflicting data and retry this step.

```javascript
db.runCommand({
  collMod: "flights",
  index: {
    name: "route_no_1_scheduled_departure_1_unique",
    unique: true
  }
})
```

Do not combine `prepareUnique` and `unique` in the same request. After successful conversion, the index is reported as unique and enforces uniqueness for the indexed key combination.

## Appendix: Ordered-index option

Ordered/composite indexing is enabled by default, so normal index creation does not require `enableOrderedIndex`. When needed, the option is accepted directly in an index specification or inside `storageEngine`.

| Parameter | Description |
| --- | --- |
| **`enableOrderedIndex`** | (Optional) A boolean or integer controlling ordered/composite indexing. Numeric values must be `-1`, `1`, or `2`; booleans are preferred. |

To explicitly request an ordered index alongside existing indexes on `flights`, give it a distinct name and set `storageEngine.enableOrderedIndex`:

```javascript
db.flights.createIndex(
  { status: 1 },
  {
    name: "flight_status_ordered",
    storageEngine: { enableOrderedIndex: true }
  }
)
```

This requests ordered/composite indexing explicitly rather than relying on the default.

To verify the index configuration, use `listIndexes` through the shell's `getIndexes()` helper and find the named index:

```javascript
db.flights.getIndexes()
```

Find `flight_status_ordered` in the returned list and check that its index specification includes `storageEngine.enableOrderedIndex: true`.

See [wire protocol and common fields](../diagnostic/wire-protocol.md). Source: `pg_documentdb/src/commands/create_indexes.c`, `ParseCreateIndexesArg`, `ParseIndexDefDoc`, `ParseCustomIndexDefOption`, and `pg_documentdb/src/vector/vector_index_kind_impl.c`; uniqueness conversion: `pg_documentdb/src/commands/coll_mod.c`, `HandleUniqueConversion` and `UpdatePostgresIndexesForUnique`.
