---
title: insert
description: The insert command in DocumentDB creates new documents in a collection
type: commands
category: query-and-write
---

# insert

The `insert` command is used to create new documents into a collection. Either a single document or multiple documents can be inserted in one go.

Inserts a batch of documents into a collection. Documents without `_id` receive generated identifiers; an explicit `_id` must be unique within the collection.

## Parameters

| Parameter | Description |
| --- | --- |
| **`insert`** | (Required) A string specifying the collection name. |
| **`documents`** | (Required) An array of documents specifying the batch. Drivers can instead send an OP_MSG document sequence named `documents`; do not provide both representations. |
| **`ordered`** | (Optional) A boolean that defaults to `true`: stop after the first write error. `false` attempts the remaining documents. |
| **`bypassDocumentValidation`** | (Optional) A boolean that bypasses schema validation when the server enables that capability; otherwise ignored. |
| **`maxTimeMS`** | (Optional) A number specifying the execution timeout in milliseconds. |
| **`comment`** | (Optional) A string carrying trace context for gateway request tracing; see [common request fields](../diagnostic/wire-protocol.md). |

`collation` is rejected by default.

## Example: load the airline dataset

The [airline dataset setup](../diagnostic/wire-protocol.md#airline-example-dataset) uses `insert` to load four flights. It supplies explicit identifiers so that the read examples have reproducible results.

## Example: add an airport

Run this in the `bookings` database, with no airport document using `_id: "SVO"`. The airport code serves as the document identifier:

```javascript
db.airports.insertOne({
    _id: "SVO",
    airport_name: "Sheremetyevo",
    city: "Moscow",
    country: "Russia",
    timezone: "Europe/Moscow"
}, { maxTimeMS: 5000 })
```

On success, the helper reports `acknowledged: true` and `insertedId: "SVO"`. The underlying wire-command response counts the insert in `n`.

## Example: continue after a duplicate identifier

Run this after the preceding airport insert, with no airport using `_id: "LED"`:

```javascript
db.airports.insertMany(
  [
    { _id: "SVO", airport_name: "Sheremetyevo", city: "Moscow", country: "Russia", timezone: "Europe/Moscow" },
    { _id: "LED", airport_name: "Pulkovo", city: "Saint Petersburg", country: "Russia", timezone: "Europe/Moscow" }
  ],
  { ordered: false }
)
```

The first entry produces a duplicate-key error, and the second can still be inserted. Mongosh throws a bulk-write error that identifies the failed input entry by `index: 0`. The underlying command response uses `writeErrors` for failures and `n` for successful inserts. With `ordered: true`, processing would stop at the duplicate instead.

## Shell syntax

```javascript
db.collection.insert(
   <single document or array of documents>,
   {
     writeConcern: <document>,
     ordered: <boolean>
   }
)
```

## Detailed Examples

### Inserting a single document

The following command inserts a single document into the stores collection in the StoreData database.

```javascript
db.stores.insertOne({
  "storeId": "12345",
  "name": "Boulder Innovations",
  "location": {
    "lat": 37.7749,
    "lon": -122.4194
  },
  "staff": {
    "totalStaff": {
      "fullTime": 15,
      "partTime": 10
    }
  },
  "sales": {
    "totalSales": 500000.00,
    "salesByCategory": [
      {
        "categoryName": "Laptops",
        "totalSales": 300000.00
      },
      {
        "categoryName": "Smartphones",
        "totalSales": 200000.00
      }
    ]
  },
  "promotionEvents": [
    {
      "eventName": "Summer Sale",
      "promotionalDates": {
        "startDate": "2024-06-01",
        "endDate": "2024-06-30"
      },
      "discounts": [
        {
          "categoryName": "Laptops",
          "discountPercentage": 15
        },
        {
          "categoryName": "Smartphones",
          "discountPercentage": 10
        }
      ]
    },
    {
      "eventName": "Holiday Specials",
      "promotionalDates": {
        "startDate": "2024-12-01",
        "endDate": "2024-12-31"
      },
      "discounts": [
        {
          "categoryName": "Laptops",
          "discountPercentage": 20
        },
        {
          "categoryName": "Smartphones",
          "discountPercentage": 25
        }
      ]
    }
  ]
})
```

### Inserting multiple documents

The following command inserts an array of documents into the stores collection.

```javascript
db.stores.insertMany([
  {
    "storeId": "12346",
    "name": "Graphic Design Institute",
    "location": {
      "lat": 34.0522,
      "lon": -118.2437
    },
    "staff": {
      "totalStaff": {
        "fullTime": 20,
        "partTime": 5
      }
    },
    "sales": {
      "totalSales": 750000.00,
      "salesByCategory": [
        {
          "categoryName": "Laptops",
          "totalSales": 400000.00
        },
        {
          "categoryName": "Smartphones",
          "totalSales": 350000.00
        }
      ]
    },
    "promotionEvents": [
      {
        "eventName": "Black Friday",
        "promotionalDates": {
          "startDate": "2024-11-25",
          "endDate": "2024-11-30"
        },
        "discounts": [
          {
            "categoryName": "Laptops",
            "discountPercentage": 25
          },
          {
            "categoryName": "Smartphones",
            "discountPercentage": 30
          }
        ]
      }
    ]
  },
  {
    "storeId": "12347",
    "name": "Relecloud",
    "location": {
      "lat": 40.7128,
      "lon": -74.0060
    },
    "staff": {
      "totalStaff": {
        "fullTime": 10,
        "partTime": 20
      }
    },
    "sales": {
      "totalSales": 600000.00,
      "salesByCategory": [
        {
          "categoryName": "Laptops",
          "totalSales": 350000.00
        },
        {
          "categoryName": "Smartphones",
          "totalSales": 250000.00
        }
      ]
    },
    "promotionEvents": [
      {
        "eventName": "New Year Sale",
        "promotionalDates": {
          "startDate": "2024-01-01",
          "endDate": "2024-01-07"
        },
        "discounts": [
          {
            "categoryName": "Laptops",
            "discountPercentage": 10
          },
          {
            "categoryName": "Smartphones",
            "discountPercentage": 15
          }
        ]
      }
    ]
  }
])
```

### Specifying a value for the _id field

If the _id field is not specified, the server auto generates a unique ObjectId() value for the document. If the document does specify the _id field, it must be a globally unique value across all documents within the collection.

If a duplicate value for the _id field is specified, a duplicate key violation error will be thrown by the server.

```json
{
    "WriteErrors": [
        {
            "WriteError": {
                "err": {
                    "index": 0,
                    "code": 11000,
                    "errmsg": "Duplicate key violation on the requested collection: Index '_id_'",
                    "errInfo": "undefined",
                    "op": {
                        "testField": "testValue",
                        "_id": "1"
                    }
                }
            }
        }
    ]
}
```

### Inserting multiple documents in order

Documents inserted in bulk are inserted in the order provided, and the batch stops at the first failure. This is the default, so `ordered: true` below is explicit rather than required. Pass `ordered: false` instead when you want the server to attempt every document regardless of errors.

```javascript
db.stores.insertMany([
  {
    "_id": "123456",
    "storeId": "123456",
    "name": "Graphic Design Institute",
    "location": {
      "lat": 34.0522,
      "lon": -118.2437
    },
    "staff": {
      "totalStaff": {
        "fullTime": 20,
        "partTime": 5
      }
    },
    "sales": {
      "totalSales": 750000.00,
      "salesByCategory": [
        {
          "categoryName": "Laptops",
          "totalSales": 400000.00
        },
        {
          "categoryName": "Smartphones",
          "totalSales": 350000.00
        }
      ]
    },
    "promotionEvents": [
      {
        "eventName": "Black Friday",
        "promotionalDates": {
          "startDate": "2024-11-25",
          "endDate": "2024-11-30"
        },
        "discounts": [
          {
            "categoryName": "Laptops",
            "discountPercentage": 25
          },
          {
            "categoryName": "Smartphones",
            "discountPercentage": 30
          }
        ]
      }
    ]
  },
  {
    "_id": "234567",
    "storeId": "234567",
    "name": "Relecloud",
    "location": {
      "lat": 40.7128,
      "lon": -74.0060
    },
    "staff": {
      "totalStaff": {
        "fullTime": 10,
        "partTime": 20
      }
    },
    "sales": {
      "totalSales": 600000.00,
      "salesByCategory": [
        {
          "categoryName": "Laptops",
          "totalSales": 350000.00
        },
        {
          "categoryName": "Smartphones",
          "totalSales": 250000.00
        }
      ]
    },
    "promotionEvents": [
      {
        "eventName": "New Year Sale",
        "promotionalDates": {
          "startDate": "2024-01-01",
          "endDate": "2024-01-07"
        },
        "discounts": [
          {
            "categoryName": "Laptops",
            "discountPercentage": 10
          },
          {
            "categoryName": "Smartphones",
            "discountPercentage": 15
          }
        ]
      }
    ]
  }
], { ordered: true })
```

A successful insert returns the ids of the inserted documents, keyed by their position in the input array. Note that `insertedIds` reports input positions, not execution order, so its shape is the same under `ordered: false` — it is not a way to confirm the order in which documents were applied:

```json
{
    "acknowledged": true,
    "insertedIds": {
        "0": "123456",
        "1": "234567"
    }
}
```

## Related content

- [update with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/update/)
- [find with DocumentDB](https://documentdb.io/docs/reference/commands/query-and-write/find/)

## Implementation

Source: `pg_documentdb/src/commands/insert.c`, `BuildBatchInsertionSpec`. See [wire protocol and common fields](../diagnostic/wire-protocol.md) for transaction and retryable-write metadata.
