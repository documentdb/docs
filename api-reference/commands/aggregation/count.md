---
title: Count
description: Count command is used to count the number of documents in a collection that match a specified query.
type: commands
category: aggregation
---

# count

The `count` command is used to count the number of documents in a collection that match a specified query. This command is useful for obtaining quick statistics about the data stored in your collections, such as the number of documents that meet certain criteria.

Returns a count in `n`, rather than a cursor. An omitted or empty query without other query modifiers can use collection statistics; supplying a query or hint selects a query-based counting path.

## Parameters

| Parameter | Description |
| --- | --- |
| **`count`** | (Required) A string specifying the collection name. |
| **`query`** | (Optional) A document specifying the match filter. Omit it to count the collection without a filter. |
| **`skip`** | (Optional) A non-negative integer that skips this many matches before counting. |
| **`limit`** | (Optional) An integer specifying the maximum count; `0` means no limit. Negative values use their absolute value. |
| **`hint`** | (Optional) A string or document specifying the index name or key pattern; forces the query-based counting path. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`collation`** | (Optional) A document specifying the comparison rules; requires `documentdb_core.enableCollation`. |

## Example: count delayed flights

Using the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset):

```javascript
db.flights.count({ status: "Delayed" })
```

The helper returns `2`. Unlike the raw command response, this is a number rather than a document containing `n`.

To count at most one matching flight after skipping the first match:

```javascript
db.flights.count({ departure_airport: "SVO" }, { skip: 1, limit: 1 })
```

This returns `1`. The `count()` helper targets the command documented here; current mongosh marks that helper as deprecated.

## Shell syntax

```javascript
db.collection.count(query, options)
```

## Detailed Examples

Here are some examples to demonstrate the usage of the `count` command:

### Example 1. Counting all documents in a collection

To count all documents in the `stores` collection:

```javascript
db.stores.count({})
```

#### Sample output

```json
60570
```

### Example 2. Counting documents with specific criteria

To count the number of stores with a specific `_id` store ID:

```javascript
db.stores.count({ "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926" })
```

#### Sample output

```json
1
```

### Example 3. Counting documents with nested criteria

To count the number of stores that have a specific promotion event:

```javascript
db.stores.count({ "promotionEvents.eventName": "Incredible Discount Days" })   
```

#### Sample output

```json
2156
```

### Example 4. Counting documents with multiple criteria

To count the number of stores located at a specific latitude and longitude:

```javascript
db.stores.count({ "location.lat": -2.4111, "location.lon": 72.1041 })
```

#### Sample output

```json
1
```

## Implementation

Source: `pg_documentdb/src/aggregation/bson_aggregation_pipeline.c`, `GenerateCountQuery`. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for source scope and shared metadata.
