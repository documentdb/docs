---
title: update
description: The update commands in DocumentDB modify documents within a collection that match specific filters
type: commands
category: query-and-write
---

# update

The `update` command is used to modify existing documents within a collection. The `update` command can be used to update one or multiple documents based on filtering criteria. Values of fields can be changed, new fields and values can be added and existing fields can be removed.

Applies a batch of updates. Each entry supplies its own filter and modification, allowing one-document updates, multi-document updates, or upserts.

## Parameters

| Parameter | Description |
| --- | --- |
| **`update`** | (Required) A string specifying the collection name. |
| **`updates`** | (Required) An array of documents specifying the operations. Drivers can instead send an OP_MSG sequence named `updates`, but not both. |
| **`ordered`** | (Optional) A boolean that defaults to `true`; stop after the first operation error. |
| **`let`** | (Optional) A document specifying the command-scoped expression variables referenced as `$$name`. |
| **`bypassDocumentValidation`** | (Optional) A boolean that bypasses schema validation when the server enables that capability; otherwise ignored. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

## Per-update parameters

| Parameter | Description |
| --- | --- |
| **`q`** | (Required) A document specifying the match filter. |
| **`u`** | (Required) A replacement or operator document, or an array defining an update pipeline. |
| **`multi`** | (Optional) A boolean that updates all matches if `true`; defaults to `false`. |
| **`upsert`** | (Optional) A boolean that inserts if no document matches; defaults to `false`. |
| **`arrayFilters`** | (Optional) An array of documents that selects named positional identifiers in operator updates; incompatible with pipeline updates. |
| **`sort`** | (Optional) A document that selects the first matching document in this order; cannot be combined with `multi: true`. |
| **`collation`** | (Optional) A document specifying the comparison rules; requires `documentdb_core.enableCollation` and cluster version 1.1 or later. |

## Example: record a revised delay

Each example below is independent and starts from the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset).

```javascript
db.flights.updateOne(
  { route_no: "PG0102" },
  { $inc: { delay_minutes: 3 } },
  { upsert: false, maxTimeMS: 5000 }
)
```

The flight on route `PG0102` changes from a 12-minute delay to a 15-minute delay. The helper reports `matchedCount: 1` and `modifiedCount: 1`, corresponding to the wire response's `n` and `nModified`.

## Example: change a fare class inside an array

```javascript
db.flights.updateOne(
  { route_no: "PG0101" },
  { $set: { "segments.$[segment].fare_conditions": "Business" } },
  { arrayFilters: [{ "segment.ticket_no": "0005432000101" }] }
)
```

Only ticket segment `0005432000101` in the flight on route `PG0101` changes from `Economy` to `Business`. Other ticket segments and flights are unchanged.

## Example: update every delayed flight

```javascript
db.flights.updateMany(
  { status: "Delayed" },
  { $set: { dispatch_reviewed: true } }
)
```

The new field is added to `PG0102` and `PG0104`. Do not add a per-update `sort` to this multi-document update.

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

### Example 1 - Update a single document using the $inc operator

Increment the totalSales by 10 and decrement the number of full time staff for a document with the specified _id.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$inc": {"sales.salesByCategory.0.totalSales": 10, "staff.totalStaff.fullTime": -6}})
```

### Example 2 - Update a single document using the $min operator

Update the totalStaff count for the document with the specified _id to 10 if the current value of the field is greater than 10.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$min": {"staff.totalStaff.fullTime": 10}})
```

### Example 3 - Update a single document using the $max operator

Update the totalStaff count for the document with the specified _id to 14 if the current value of the field is less than 14.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$max": {"staff.totalStaff.fullTime": 14}})
```

### Example 4 - Update a single document using the $mul operator

Multiple the count of part time employees by 2 for the document with the specified _id value.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$mul": {"staff.totalStaff.partTime": 2}})
```

### Example 5 - Update a single document using the $rename operator

Rename the totalSales and totalStaff fields to fullSales and staffCounts respectively.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$rename": {"sales.totalSales": "sales.fullSales", "staff.totalStaff": "staff.staffCounts"}})
```

### Example 6 - Update a single document using the $set operator

Set the fullSales field to 3700 for the document with the specified _id value.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$set": {"sales.fullSales": 3700}})
```

### Example 7 - Update a single document using the $unset operator

Remove the lon field from the location object in the document with the specified _id value.

```javascript
db.stores.updateOne({"_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4"}, {"$unset": {"location.lon": ""}})
```

### Example 8 - Update multiple documents

Update all documents where the first promotional event starts in February to start in March.

```javascript
db.stores.updateMany({"promotionEvents.0.promotionalDates.startDate.Month": 2}, {"$inc": {"promotionEvents.0.promotionalDates.startDate.Month": 1}})
```

### Example 9 - Upsert a single document

Set the upsert flag to true to create a new document if the document specified in the query filter does not exist in the collection.

```javascript
db.stores.updateOne({"_id": "NonExistentDocId"}, {"$set": {"name": "Lakeshore Retail", "sales.totalSales": 0}}, {"upsert": true})
```

## Related content

- [insert with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/insert/)
- [delete with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/delete/)

## Implementation

Source: `pg_documentdb/src/commands/update.c`, `BuildBatchUpdateSpec` and per-update parsing. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for session and transaction metadata.
