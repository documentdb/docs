---
title: $replaceRoot
description: The $replaceRoot stage replaces the input document with the specified document.
type: operators
category: aggregation
---

# $replaceRoot

The `$replaceRoot` stage replaces the input document with the specified document. The operation replaces all existing fields in the input document, including the `_id` field. This is useful for promoting an embedded document to the top level.

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
| **`newRoot`** | Required. A document expression that resolves to a document. The expression can be any valid expression that resolves to a document, such as a field path to an embedded document, a `$mergeObjects` expression, or a literal document. |

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
      { "categoryName": "Wine Accessories", "totalSales": 34440 }
    ]
  }
}
```

### Example 1: Promote an embedded document

Promote the `staff.totalStaff` subdocument to the top level:

```javascript
db.stores.aggregate([
  { $replaceRoot: { newRoot: "$staff.totalStaff" } },
  { $limit: 1 }
])
```

This query returns:

```json
[
  { "fullTime": 8, "partTime": 20 }
]
```

### Example 2: Use $mergeObjects to combine fields

Merge the staff subdocument with additional top-level fields:

```javascript
db.stores.aggregate([
  {
    $replaceRoot: {
      newRoot: {
        $mergeObjects: [
          "$staff.totalStaff",
          { storeName: "$name", totalSales: "$sales.totalSales" }
        ]
      }
    }
  },
  { $limit: 1 }
])
```

This query returns:

```json
[
  {
    "fullTime": 8,
    "partTime": 20,
    "storeName": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalSales": 75670
  }
]
```

### Example 3: Replace root after $unwind

Extract individual sales categories as top-level documents:

```javascript
db.stores.aggregate([
  { $unwind: "$sales.salesByCategory" },
  {
    $replaceRoot: {
      newRoot: {
        $mergeObjects: [
          "$sales.salesByCategory",
          { storeName: "$name" }
        ]
      }
    }
  },
  { $limit: 3 }
])
```

## Limitations

- If `newRoot` evaluates to a missing value or a non-document type, the operation errors

## Key Takeaways

- **Replaces the entire document** — the output document is the evaluated `newRoot` expression
- **`$replaceWith` is an alias** — `{ $replaceWith: <expression> }` is shorthand for `{ $replaceRoot: { newRoot: <expression> } }`
- **Combine with `$mergeObjects`** — use `$mergeObjects` to preserve fields from the original document while promoting an embedded document
