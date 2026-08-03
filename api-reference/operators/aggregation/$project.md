---
title: $project
description: The $project stage in the aggregation pipeline reshapes documents by including, excluding, or computing fields.
type: operators
category: aggregation
---

# $project

The `$project` stage in the aggregation pipeline reshapes each document that passes through it. Use it to include only the fields you need, remove fields you do not, rename fields, or add fields computed from existing values.

## Syntax

```javascript
{
  $project: {
    <field1>: <1 | true | 0 | false | expression>,
    <field2>: <1 | true | 0 | false | expression>,
    ...
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`field`** | The name of a field to include, exclude, or compute. Dotted paths such as `sales.totalSales` address nested fields. |
| **`1` or `true`** | Includes the field in the output document. |
| **`0` or `false`** | Excludes the field from the output document. |
| **`expression`** | An aggregation expression whose result becomes the value of the field. Adding a computed field counts as an inclusion. |

## Examples

The examples on this page use the following document from the `stores` collection.

```json
{
  "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
  "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
  "location": { "lat": -89.2384, "lon": -46.4012 },
  "staff": { "totalStaff": { "fullTime": 8, "partTime": 20 } },
  "sales": {
    "totalSales": 75670,
    "salesByCategory": [
      { "categoryName": "Wine Accessories", "totalSales": 34440 },
      { "categoryName": "Bitters", "totalSales": 39496 },
      { "categoryName": "Rum", "totalSales": 1734 }
    ]
  },
  "tags": ["Wine", "Bitters"]
}
```

### Example 1: Including specific fields

This query keeps only the store name and its total sales, and suppresses `_id`.

```javascript
db.stores.aggregate([
  { $project: { _id: 0, name: 1, "sales.totalSales": 1 } }
])
```

This query returns the following result:

```json
[
  {
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "sales": { "totalSales": 75670 }
  }
]
```

Projecting a dotted path preserves the surrounding structure — `sales.totalSales` comes back nested inside `sales`, not flattened.

### Example 2: Adding a computed field

This query returns the store name alongside a headcount computed from the two staff fields.

```javascript
db.stores.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      totalStaff: {
        $add: ["$staff.totalStaff.fullTime", "$staff.totalStaff.partTime"]
      }
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "totalStaff": 28
  }
]
```

### Example 3: Excluding fields

When every field in the specification is set to `0`, the stage is an exclusion projection: every field except the listed ones is kept.

```javascript
db.stores.aggregate([
  { $project: { sales: 0, staff: 0, location: 0, tags: 0 } }
])
```

This query returns the following result:

```json
[
  {
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth"
  }
]
```

## Behavior

**`_id` is included by default.** It is the only field you can exclude from an inclusion projection; suppress it with `_id: 0`. Using `{ $project: { _id: 0 } }` on its own is a pure exclusion and keeps every other field.

**Inclusion and exclusion cannot be mixed.** Apart from top-level `_id`, a single `$project` is either an inclusion projection or an exclusion projection. Mixing them fails:

```javascript
db.stores.aggregate([{ $project: { name: 1, tags: 0 } }])
```

```
exclusion cannot be applied to field tags within the inclusion projection.
```

**Field names cannot begin with `$`.** A literal field name starting with `$` is rejected — use `$getField` or `$setField` for such fields:

```
FieldPath field names cannot begin with the operators symbol '$'; you might want to use $getField or $setField instead.
```

**An empty specification is accepted.** `{ $project: {} }` passes documents through unchanged rather than raising an error.

## Related

- [`$addFields`](../%24addfields/) — adds fields while keeping all existing ones.
- [`$set`](../%24set/) — alias for `$addFields`.
- [`$unset`](../%24unset/) — removes fields without switching the whole stage to exclusion semantics.
