---
title: $max
description: The $max operator updates the field to the specified value if the specified value is greater than the current value.
type: operators
category: field-update
---

# $max

The `$max` operator updates the value of a field to the specified value if the specified value is greater than the current value of the field. If the field does not exist, `$max` creates the field and sets it to the specified value. The `$max` operator compares values using BSON comparison order.

## Syntax

```javascript
{
  $max: {
    <field1>: <value1>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field`** | The name of the field to update. Use dot notation for nested fields. |
| **`value`** | The value to compare against the current field value. The field is updated only if this value is greater than the current value. |

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

### Example 1: Updating when the specified value is greater

When the specified value is greater than the current value, `$max` updates the field. Since 50000 is greater than the current `totalSales` of 35150, the field is updated.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $max: {
      "sales.totalSales": 50000
    }
  }
)
```

After this operation, the document is updated as follows:

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
        "totalSales": 50000
    }
}
```

### Example 2: No update when the specified value is less

When the specified value is less than the current value, `$max` has no effect. Since 20000 is less than the current `totalSales` of 35150, the field remains unchanged.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $max: {
      "sales.totalSales": 20000
    }
  }
)
```

The document remains unchanged after this operation.

### Example 3: Creating a new field

If the field does not exist, `$max` creates it and sets it to the specified value.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $max: {
      "sales.highestDailySales": 5000
    }
  }
)
```

After this operation, the document includes the new field:

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
        "totalSales": 35150,
        "highestDailySales": 5000
    }
}
```
