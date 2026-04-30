---
title: $replaceRoot
description: The $replaceRoot stage in the aggregation pipeline replaces the input document with the specified document.
type: operators
category: aggregation
---

# $replaceRoot

The `$replaceRoot` stage in the aggregation pipeline replaces the input document with the specified document. The stage promotes a sub-document or a computed document to the top level, effectively replacing the entire content of each input document. This is useful for restructuring documents or extracting embedded documents to the top level.

> **Note:** `$replaceRoot` is functionally equivalent to `$replaceWith`. See the [`$replaceWith`]($replacewith.md) documentation for the alias syntax.

## Syntax

```javascript
{
  $replaceRoot: {
    newRoot: <expression>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`newRoot`** | Required. An expression that resolves to a document. The expression must evaluate to an object; if the expression resolves to a missing or non-object value, the stage produces an error. |

## Examples

Consider this sample document from the stores collection.

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

### Example 1: Promoting a sub-document to the top level

To replace each document with its `sales` sub-document:

```javascript
db.stores.aggregate([
  {
    $replaceRoot: {
      newRoot: "$sales"
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
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
  {
    "totalSales": 47510,
    "salesByCategory": [
      {
        "categoryName": "DJ Speakers",
        "totalSales": 25000
      },
      {
        "categoryName": "Turntables",
        "totalSales": 22510
      }
    ]
  }
]
```

### Example 2: Creating a new document with computed fields

To replace each document with a new document that merges original fields with computed values:

```javascript
db.stores.aggregate([
  {
    $replaceRoot: {
      newRoot: {
        $mergeObjects: [
          { storeId: "$_id", storeName: "$name" },
          "$location",
          {
            totalStaff: {
              $add: ["$staff.totalStaff.fullTime", "$staff.totalStaff.partTime"]
            }
          }
        ]
      }
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
  {
    "storeId": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "storeName": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "lat": -89.2384,
    "lon": -46.4012,
    "totalStaff": 28
  },
  {
    "storeId": "7954bd5c-9ac2-4c10-bb7a-2b79bd0963c5",
    "storeName": "Lakeshore Retail | DJ Equipment Stop - Port Cecile",
    "lat": 45.1234,
    "lon": -93.5678,
    "totalStaff": 15
  }
]
```

### Example 3: Promoting unwound array elements to the top level

To unwind the promotion events and replace each document with the promotion event details:

```javascript
db.stores.aggregate([
  {
    $unwind: "$promotionEvents"
  },
  {
    $replaceRoot: {
      newRoot: {
        $mergeObjects: [
          "$promotionEvents",
          { storeName: "$name" }
        ]
      }
    }
  },
  { $limit: 2 }
])
```

The first two results returned by this query are:

```json
[
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
    ],
    "storeName": "First Up Consultants | Beverage Shop - Satterfieldmouth"
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
    ],
    "storeName": "First Up Consultants | Beverage Shop - Satterfieldmouth"
  }
]
```
