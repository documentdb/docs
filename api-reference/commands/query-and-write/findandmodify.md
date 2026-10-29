---
title: findAndModify
description: The findAndModify command is used to atomically modify and return a single document.
type: commands
category: query-and-write
---

# findAndModify

The `findAndModify` command is used to atomically modify and return a single document. This command is useful for operations that require reading and updating a document in a single step, ensuring data consistency. Common use cases include implementing counters, queues, and other atomic operations.

## Parameters

| Parameter | Description |
| --- | --- |
| **`findAndModify`** | (Required) A string specifying the collection name. |
| **`query`** | (Optional) A document specifying the selection filter. |
| **`sort`** | (Optional) A document specifying the ordering used to select one matching document. |
| **`update`** | (Required for updates) A replacement or operator document, or an array defining an update pipeline. |
| **`remove`** | (Optional) A boolean that selects the removal path instead of an update when `true`. |
| **`new`** | (Optional) A boolean that returns the post-update document if `true`; otherwise return the original. |
| **`fields`** | (Optional) A document specifying the projection applied to the returned document. |
| **`upsert`** | (Optional) A boolean that inserts if the update filter matches no document. |
| **`arrayFilters`** | (Optional) An array of documents specifying the filters for named positional identifiers in operator updates; incompatible with pipeline updates. Not applied to removal. |
| **`let`** | (Optional) A document specifying the command-scoped variables referenced as `$$name`. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`bypassDocumentValidation`** | (Optional) A boolean that is applied only when schema validation and validation bypass are enabled; otherwise ignored. |
| **`collation`** | (Optional) A document specifying the comparison rules; requires `documentdb_core.enableCollation` and cluster version 1.1 or later. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

Specify either an update or `remove: true`, not both. Removal cannot request `new: true` or `upsert: true`.

## Example: revise and return a departure

Each example below is independent and starts from the [airline example dataset](../diagnostic/wire-protocol.md#airline-example-dataset).

```javascript
db.flights.findOneAndUpdate(
  { route_no: "PG0102" },
  { $set: { delay_minutes: 15 } },
  {
    returnDocument: "after",
    projection: { _id: 0, route_no: 1, delay_minutes: 1 }
  }
)
```

The helper returns the selected document directly:

```json
{ "route_no": "PG0102", "delay_minutes": 15 }
```

`returnDocument: "after"` maps to the wire field `new: true`, and `projection` maps to `fields`. Without requesting the post-update document, the returned value would contain the original delay of `12`.

## Example: remove the earliest delayed service

```javascript
db.flights.findOneAndDelete(
  { status: "Delayed" },
  {
    sort: { scheduled_departure: 1, _id: 1 },
    projection: { _id: 0, route_no: 1 }
  }
)
```

This sends the removal form of `findAndModify`, removes `PG0102`, and returns `{ route_no: "PG0102" }` directly. Unlike a `delete` entry, this path can use `sort` to choose a particular match.

## Shell syntax

```javascript
db.collection.findAndModify({
   query: <document>,
   sort: <document>,
   remove: <boolean>,
   update: <document>,
   new: <boolean>,
   fields: <document>,
   upsert: <boolean>
})
```

## Detailed Examples

### Example 1: Update total sales

Suppose we want to update the total sales for the store with `_id` "e5767a9f-cd95-439c-9ec4-7ddc13d22926" to `550000.00` and return the updated document.

```javascript
db.stores.findAndModify({
   query: { "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926" },
   update: { $set: { "sales.totalSales": 550000.00 } },
   new: true
})
```

#### Sample output

```json
{
  "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926",
  "name": "Marina's Eyewear Bargains",
  "location": { "lat": -87.4376, "lon": 42.2928 },
  "staff": { "totalStaff": { "fullTime": 20, "partTime": 6 } },
  "sales": {
    "totalSales": 550000,
    "salesByCategory": [
      { "categoryName": "Round Sunglasses", "totalSales": 39621 },
      { "categoryName": "Reading Glasses", "totalSales": 1146 },
      { "categoryName": "Aviators", "totalSales": 9385 }
    ]
  },
  "promotionEvents": [
    {
      "eventName": "Incredible Discount Days",
      "promotionalDates": {
        "startDate": { "Year": 2024, "Month": 2, "Day": 11 },
        "endDate": { "Year": 2024, "Month": 2, "Day": 18 }
      },
      "discounts": [
        { "categoryName": "Square Sunglasses", "discountPercentage": 16 },
        { "categoryName": "Safety Glasses", "discountPercentage": 17 },
        { "categoryName": "Wayfarers", "discountPercentage": 7 },
        { "categoryName": "Eyewear Accessories", "discountPercentage": 12 }
      ]
    }
],
  "tag": [
    "#ShopLocal",
    "#FashionStore",
    "#SeasonalSale",
    "#FreeShipping",
    "#MembershipDeals"
  ]
}
```

### Example 2: Add a new promotional event

Let's add a new promotional event called "Electronics Super Saver" to the store with `_id_` "e5767a9f-cd95-439c-9ec4-7ddc13d22926" and return the updated document.

```javascript
db.stores.findAndModify({
   query: { "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926" },
   update: { $push: { "promotionEvents": {
       "eventName": "Electronics Super Saver",
       "promotionalDates": {
         "startDate": "2025-09-31",
         "endDate": "2025-09-31"
       },
       "discounts": [
         {
           "categoryName": "Laptops",
           "discountPercentage": 45
         },
         {
           "categoryName": "Smartphones",
           "discountPercentage": 25
         }
       ]
   }}},
   new: true
})
```

#### Sample output

```json
{
  "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926",
  "name": "Marina's Eyewear Bargains",
  "location": { "lat": -87.4376, "lon": 42.2928 },
  "staff": { "totalStaff": { "fullTime": 20, "partTime": 6 } },
  "sales": {
    "totalSales": 550000,
    "salesByCategory": [
      { "categoryName": "Round Sunglasses", "totalSales": 39621 },
      { "categoryName": "Reading Glasses", "totalSales": 1146 },
      { "categoryName": "Aviators", "totalSales": 9385 }
    ]
  },
  "promotionEvents": [
    {
      "eventName": "Incredible Discount Days",
      "promotionalDates": {
        "startDate": { "Year": 2024, "Month": 2, "Day": 11 },
        "endDate": { "Year": 2024, "Month": 2, "Day": 18 }
      },
      "discounts": [
        { "categoryName": "Square Sunglasses", "discountPercentage": 16 },
        { "categoryName": "Safety Glasses", "discountPercentage": 17 },
        { "categoryName": "Wayfarers", "discountPercentage": 7 },
        { "categoryName": "Eyewear Accessories", "discountPercentage": 12 }
      ]
    },
    {
      "eventName": "Electronics Super Saver",
      "promotionalDates": { "startDate": "2025-09-31", "endDate": "2025-09-31" },
      "discounts": [
        { "categoryName": "Laptops", "discountPercentage": 45 },
        { "categoryName": "Smartphones", "discountPercentage": 25 }
      ]
    }
  ],
  "tag": [
    "#ShopLocal",
    "#FashionStore",
    "#SeasonalSale",
    "#FreeShipping",
    "#MembershipDeals"
  ]
}
```

### Example 3: Remove a promotional event

Suppose we want to remove the "Electronics Super Saver" promotional event from the store with `_id` "e5767a9f-cd95-439c-9ec4-7ddc13d22926" and return the updated document.

```javascript
db.stores.findAndModify({
   query: { "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926" },
   update: { $pull: { "promotionEvents": { "eventName": "Electronics Super Saver" } } },
   new: true
})
```

#### Sample output

```json
{
  "_id": "e5767a9f-cd95-439c-9ec4-7ddc13d22926",
  "name": "Marina's Eyewear Bargains",
  "location": { "lat": -87.4376, "lon": 42.2928 },
  "staff": { "totalStaff": { "fullTime": 20, "partTime": 6 } },
  "sales": {
    "totalSales": 550000,
    "salesByCategory": [
      { "categoryName": "Round Sunglasses", "totalSales": 39621 },
      { "categoryName": "Reading Glasses", "totalSales": 1146 },
      { "categoryName": "Aviators", "totalSales": 9385 }
    ]
  },
  "promotionEvents": [
    {
      "eventName": "Incredible Discount Days",
      "promotionalDates": {
        "startDate": { "Year": 2024, "Month": 2, "Day": 11 },
        "endDate": { "Year": 2024, "Month": 2, "Day": 18 }
      },
      "discounts": [
        { "categoryName": "Square Sunglasses", "discountPercentage": 16 },
        { "categoryName": "Safety Glasses", "discountPercentage": 17 },
        { "categoryName": "Wayfarers", "discountPercentage": 7 },
        { "categoryName": "Eyewear Accessories", "discountPercentage": 12 }
      ]
    }
  ],
  "tag": [
    "#ShopLocal",
    "#FashionStore",
    "#SeasonalSale",
    "#FreeShipping",
    "#MembershipDeals"
  ]
}
```

## Implementation

The alias `findandmodify` reaches the same wire handler.

Source: `pg_documentdb/src/commands/find_and_modify.c`, `ParseFindAndModifyMessage`. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for shared metadata.
