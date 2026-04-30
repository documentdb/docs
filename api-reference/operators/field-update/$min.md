---
title: $min
description: The $min operator updates the field to the specified value if the specified value is less than the current value.
type: operators
category: field-update
---

# $min

The `$min` operator updates the value of a field to the specified value if the specified value is less than the current value of the field. If the field does not exist, `$min` creates the field and sets it to the specified value. The `$min` operator compares values using BSON comparison order.

## Syntax

```javascript
{
  $min: {
    <field1>: <value1>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field`** | The name of the field to update. Use dot notation for nested fields. |
| **`value`** | The value to compare against the current field value. The field is updated only if this value is less than the current value. |

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

### Example 1: Updating when the specified value is less

When the specified value is less than the current value, `$min` updates the field. Since 30000 is less than the current `totalSales` of 35150, the field is updated.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $min: {
      "sales.totalSales": 30000
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
        "totalSales": 30000
    }
}
```

### Example 2: No update when the specified value is greater

When the specified value is greater than the current value, `$min` has no effect. Since 50000 is greater than the current `totalSales` of 35150, the field remains unchanged.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $min: {
      "sales.totalSales": 50000
    }
  }
)
```

The document remains unchanged after this operation.

### Example 3: Creating a new field

If the field does not exist, `$min` creates it and sets it to the specified value.

```javascript
db.stores.updateOne(
  { "_id": "40d6f4d7-50cd-4929-9a07-0a7a133c2e74" },
  {
    $min: {
      "sales.lowestDailySales": 250
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
        "lowestDailySales": 250
    }
}
```
