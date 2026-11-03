---
title: delete
description: The delete command in DocumentDB deletes documents that match a specified criteria
type: commands
category: query-and-write
---

# delete

The `delete` command is used to remove documents from a collection. A single document or multiple documents can be deleted based on a specified query filter.

Removes documents selected by each entry in a deletion batch. The per-entry `limit`, not a command-level option, determines whether one or all matches are removed.

## Parameters

| Parameter | Description |
| --- | --- |
| **`delete`** | (Required) A string specifying the collection name. |
| **`deletes`** | (Required) An array of documents specifying the operations. Drivers can instead send an OP_MSG sequence named `deletes`, but not both. |
| **`ordered`** | (Optional) A boolean that defaults to `true`; stop after the first operation error. |
| **`let`** | (Optional) A document specifying the command-scoped expression variables referenced as `$$name`. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

## Per-deletion parameters

| Parameter | Description |
| --- | --- |
| **`q`** | (Required) A document specifying the match filter. |
| **`limit`** | (Required) A number with the following behavior: `1` removes at most one match, and `0` removes every match. Fractional values and other integers are rejected. |
| **`collation`** | (Optional) A document specifying the comparison rules; requires `documentdb_core.enableCollation`. |

## Example: remove one flight service

Each example below is independent and starts from the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset).

```javascript
db.flights.deleteOne({ route_no: "PG0103" })
```

The helper reports `deletedCount: 1`, corresponding to `n: 1` in the wire response. It sends `limit: 1`. Filtering by the known service identifies the document to remove; a non-unique filter does not specify which match is chosen.

## Example: remove all delayed services

```javascript
db.flights.deleteMany({ status: "Delayed" }, { maxTimeMS: 5000 })
```

This sends `limit: 0`, removes `PG0102` and `PG0104`, and reports `deletedCount: 2`.

An empty `q` with `limit: 0` removes every document in the collection; it does not drop the collection or its indexes.

## Shell syntax

```javascript
db.collection.deleteOne(
   <filter>,
   <options>
)

db.collection.deleteMany(
   <filter>,
   <options>
)
```

## Detailed Examples

Consider this sample document from the stores collection in the StoreData database.

```json
{
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "location": {
        "lat": -89.2384,
        "lon": -46.4012
    },
    "staff": {
        "totalStaff": {
            "fullTime": 8,
            "partTime": 20
        }
    },
    "sales": {
        "totalSales": 75670,
        "salesByCategory": [
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34440
            },
            {
                "categoryName": "Bitters",
                "totalSales": 39496
            },
            {
                "categoryName": "Rum",
                "totalSales": 1734
            }
        ]
    },
    "promotionEvents": [
        {
            "eventName": "Unbeatable Bargain Bash",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 6,
                    "Day": 23
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 7,
                    "Day": 2
                }
            },
            "discounts": [
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 7
                },
                {
                    "categoryName": "Bitters",
                    "discountPercentage": 15
                },
                {
                    "categoryName": "Brandy",
                    "discountPercentage": 8
                },
                {
                    "categoryName": "Sports Drinks",
                    "discountPercentage": 22
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 19
                }
            ]
        },
        {
            "eventName": "Steal of a Deal Days",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 21
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 29
                }
            },
            "discounts": [
                {
                    "categoryName": "Organic Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "White Wine",
                    "discountPercentage": 20
                },
                {
                    "categoryName": "Sparkling Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 17
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 23
                }
            ]
        }
    ]
}
```

The sample store above runs two promotion events, and three of its discounts are at 19%, so the filter `"promotionEvents.discounts.discountPercentage": 19` matches it. Each example below is independent and assumes the collection is fully populated; they are ordered so that the destructive one comes last.

### Example 1 - Delete a document that matches a specified query filter

```javascript
db.stores.deleteOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"})
```

### Example 2 - Delete all documents that match a specified query filter

```javascript
db.stores.deleteMany({"promotionEvents.discounts.discountPercentage": 19})
```

### Example 3 - Delete only one of many documents that match a specified query filter

Use `deleteOne` rather than `deleteMany`. There is no shell option that limits `deleteMany` to a single document — `limit` is a field of the wire-protocol `deletes` array element (see below), not a `deleteMany` option, and passing it here has no effect.

```javascript
db.stores.deleteOne({"promotionEvents.discounts.discountPercentage": 19})
```

### Example 4 - Delete all documents in a collection

An empty filter matches everything, so this empties the collection:

```javascript
db.stores.deleteMany({})
```

## Wire protocol form

The shell helpers above are wrappers over the `delete` command. Each element of the `deletes` array carries its own `limit`, which must be `0` (delete every match) or `1` (delete at most one match); any other value is rejected with `The limit field in delete objects must be 0 or 1`.

```javascript
db.runCommand({
   delete: "stores",
   deletes: [
      { q: {"promotionEvents.discounts.discountPercentage": 19}, limit: 1 }
   ]
})
```

`deleteOne` sends `limit: 1`; `deleteMany` sends `limit: 0`.

## Related content

- [insert with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/insert/)
- [update with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/update/)

## Implementation

Source: `pg_documentdb/src/commands/delete.c`, `BuildBatchDeletionSpec` and per-deletion parsing. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for shared metadata.
