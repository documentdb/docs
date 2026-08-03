---
title: $unionWith
description: The $unionWith stage performs a union of two collections, combining pipeline results from two collections into a single result set.
type: operators
category: aggregation
---

# $unionWith

The `$unionWith` stage performs a union of documents from two collections. It combines the pipeline results from two collections into a single result set, similar to SQL's `UNION ALL`. The stage outputs all documents from both collections, including duplicates.

## Syntax

**String form (simple union):**

```javascript
{
  $unionWith: "<collection>"
}
```

**Document form (with pipeline):**

```javascript
{
  $unionWith: {
    coll: "<collection>",
    pipeline: [ <stage1>, <stage2>, ... ]
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`coll`** | Optional (required if `pipeline` is omitted or doesn't start with `$documents`). The name of the collection to union with. |
| **`pipeline`** | Optional. An aggregation pipeline to apply to the specified collection before the union. |

## Examples

Consider a `stores` collection and a `warehouses` collection.

**stores collection:**

```json
{
  "_id": "store-001",
  "name": "First Up Consultants | Beverage Shop",
  "city": "Seattle",
  "type": "retail"
}
```

**warehouses collection:**

```json
{
  "_id": "wh-001",
  "name": "West Coast Distribution Center",
  "city": "Portland",
  "type": "warehouse"
}
```

### Example 1: Simple union

Combine all documents from both collections:

```javascript
db.stores.aggregate([
  { $project: { name: 1, city: 1, type: 1 } },
  {
    $unionWith: "warehouses"
  }
])
```

### Example 2: Union with pipeline

Union the stores collection with a filtered and projected subset of the warehouses collection:

```javascript
db.stores.aggregate([
  { $project: { name: 1, city: 1 } },
  {
    $unionWith: {
      coll: "warehouses",
      pipeline: [
        { $match: { city: "Portland" } },
        { $project: { name: 1, city: 1 } }
      ]
    }
  }
])
```

### Example 3: Union with $documents

Use `$documents` to inject literal documents into the union:

```javascript
db.stores.aggregate([
  { $project: { name: 1, city: 1 } },
  {
    $unionWith: {
      pipeline: [
        { $documents: [
          { name: "Pop-Up Shop", city: "Austin" },
          { name: "Online Store", city: "Remote" }
        ]}
      ]
    }
  }
])
```

## Limitations

- The following stages are **not allowed** inside the `$unionWith` pipeline: `$out`, `$merge`, `$changeStream`
- If `coll` is omitted, the pipeline must start with `$documents`
- Maximum nested pipeline depth is 20 levels

## Key Takeaways

- **UNION ALL semantics** — duplicates are not removed; use `$group` after the union to deduplicate if needed
- **Schema flexibility** — the two collections do not need to have the same schema
- **Pipeline support** — apply transformations to the second collection before unioning
- **String shorthand** — use the simple string form when no pipeline transformations are needed on the second collection
