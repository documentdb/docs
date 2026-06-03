---
title: $graphLookup
description: The $graphLookup stage performs a recursive search on a collection to process graph-like or hierarchical data.
type: operators
category: aggregation
---

# $graphLookup

The `$graphLookup` stage performs a recursive search on a collection, with each iteration of the search adding to the result set from a previous iteration. This is useful for processing hierarchical or graph-structured data such as organizational charts, category trees, or social network connections.

## Syntax

```javascript
{
  $graphLookup: {
    from: <collection>,
    startWith: <expression>,
    connectFromField: <string>,
    connectToField: <string>,
    as: <string>,
    maxDepth: <number>,
    depthField: <string>,
    restrictSearchWithMatch: <document>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`from`** | Required. The target collection for the recursive search. |
| **`startWith`** | Required. Expression that specifies the value(s) to start the recursive search from. |
| **`connectFromField`** | Required. Field name whose values are recursively matched against `connectToField` in subsequent iterations. |
| **`connectToField`** | Required. Field name in the `from` collection that is matched against `connectFromField` values. |
| **`as`** | Required. Name of the array field added to each output document containing the results of the recursive search. |
| **`maxDepth`** | Optional. Non-negative integer specifying the maximum recursion depth. Default: no limit. |
| **`depthField`** | Optional. Name of a field to add to each result document indicating the recursion depth at which it was found. |
| **`restrictSearchWithMatch`** | Optional. A document specifying additional filter conditions for the recursive search. |

## Examples

Consider an `employees` collection with documents representing an organizational hierarchy.

```json
{
  "_id": 1,
  "name": "Alice",
  "role": "CEO",
  "reportsTo": null
}
{
  "_id": 2,
  "name": "Bob",
  "role": "VP Engineering",
  "reportsTo": "Alice"
}
{
  "_id": 3,
  "name": "Carol",
  "role": "Engineering Manager",
  "reportsTo": "Bob"
}
{
  "_id": 4,
  "name": "Dave",
  "role": "Senior Engineer",
  "reportsTo": "Carol"
}
```

### Example 1: Find the reporting chain

Find the full reporting chain for each employee:

```javascript
db.employees.aggregate([
  {
    $graphLookup: {
      from: "employees",
      startWith: "$reportsTo",
      connectFromField: "reportsTo",
      connectToField: "name",
      as: "reportingHierarchy"
    }
  }
])
```

### Example 2: Limit recursion depth

Find only direct and second-level reports with depth tracking:

```javascript
db.employees.aggregate([
  { $match: { name: "Alice" } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$name",
      connectFromField: "name",
      connectToField: "reportsTo",
      as: "subordinates",
      maxDepth: 2,
      depthField: "level"
    }
  }
])
```

### Example 3: Restrict search with match filter

Find the reporting chain, but only include employees with an engineering role:

```javascript
db.employees.aggregate([
  { $match: { name: "Dave" } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$reportsTo",
      connectFromField: "reportsTo",
      connectToField: "name",
      as: "engineeringChain",
      restrictSearchWithMatch: {
        role: { $regex: /Engineering/ }
      }
    }
  }
])
```

## Limitations

- The `from` collection cannot be sharded
- Maximum nested pipeline depth is 20 levels
- Field names for `connectFromField`, `connectToField`, `as`, and `depthField` cannot start with `$`

## Key Takeaways

- **Recursive graph traversal** — traverses relationships by recursively matching `connectFromField` to `connectToField`
- **Depth control** — use `maxDepth` to limit how deep the recursion goes
- **Depth tracking** — use `depthField` to annotate each result with its recursion depth
- **Filter during traversal** — use `restrictSearchWithMatch` to prune the search space
