---
title: delete
description: The delete command in DocumentDB deletes documents that match a specified criteria
type: commands
category: query-and-write
---

# delete

The `delete` command is used to remove documents from a collection. A single document or multiple documents can be deleted based on a specified query filter.

## Syntax

The basic syntax for the `delete` command is as follows:

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

### Parameters

| Parameter | Description |
| --- | --- |
| **<`filter`>** | A document that specifies the criteria for deletion. Only the documents that match the filter are deleted|
| **`options`** | Optional. A document that specifies options for the delete operation. Common options include writeConcern and collation|

## Example(s)

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

The sample store above runs two promotion events, and three of its discounts are at 19%, so the filter `"promotionEvents.discounts.discountPercentage": 19` matches it. The examples are ordered so that the destructive one comes last.

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
