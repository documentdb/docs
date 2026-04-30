---
title: $set
description: The $set operator sets the value of a field in a document.
type: operators
category: field-update
---

# $set

The `$set` operator replaces the value of a field with the specified value. If the field does not exist, `$set` creates the field and sets it to the specified value. You can use dot notation to set values for nested fields or array elements.

## Syntax

```javascript
{
  $set: {
    <field1>: <value1>,
    <field2>: <value2>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field`** | The name of the field to set. Use dot notation for nested fields or array elements. |
| **`value`** | The value to assign to the field. Can be any valid BSON type. |

## Examples

Consider this sample document from the stores collection.

```json
{
    "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74",
    "name": "Proseware, Inc. | Home Entertainment Hub - East Linwoodbury",
    "staff": {
        "totalStaff": {
            "fullTime": 14,
            "partTime": 6
        }
    },
    "sales": {
        "totalSales": 35150
    }
}
```

### Example 1: Setting top-level and nested fields

To update the store name and set the full-time staff count to a new value, use the `$set` operator on both fields.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $set: {
      "name": "Proseware, Inc. | Home Entertainment Hub - West Linwoodbury",
      "staff.totalStaff.fullTime": 18
    }
  }
)
```

### Example 2: Creating new fields

If a field does not exist, `$set` creates it with the specified value.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $set: {
      "location": {
        "lat": 33.749,
        "lon": -84.388
      },
      "sales.onlineSales": 12500
    }
  }
)
```

### Example 3: Setting array element values using dot notation

You can set specific array elements using dot notation with an index position.

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
        "totalSales": 47510,
        "salesByCategory": [
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34450
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
            "eventName": "Summer Sizzler",
            "promotionalDates": {
                "startDate": {
                    "$date": "2024-08-01T00:00:00Z"
                },
                "endDate": {
                    "$date": "2024-08-31T00:00:00Z"
                }
            },
            "discountPercentage": 15
        },
        {
            "eventName": "Oktoberfest Specials",
            "promotionalDates": {
                "startDate": {
                    "$date": "2024-10-01T00:00:00Z"
                },
                "endDate": {
                    "$date": "2024-10-31T00:00:00Z"
                }
            },
            "discountPercentage": 20
        }
    ]
}
```

Update the first promotion event's discount percentage:

```javascript
db.stores.updateOne(
  { "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4" },
  {
    $set: {
      "promotionEvents.0.discountPercentage": 25,
      "sales.salesByCategory.1.totalSales": 42000
    }
  }
)
```

### Example 4: Updating multiple documents

Use `updateMany` to set a field across all matching documents.

```javascript
db.stores.updateMany(
  { "sales.totalSales": { $lt: 50000 } },
  {
    $set: {
      "sales.performanceTier": "low"
    }
  }
)
```
