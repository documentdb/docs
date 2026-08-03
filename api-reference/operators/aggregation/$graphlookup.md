---
title: $graphLookup
description: The $graphLookup stage in the aggregation pipeline performs a recursive search across documents in a collection.
type: operators
category: aggregation
---

# $graphLookup

The `$graphLookup` stage in the aggregation pipeline performs a recursive search over a collection and attaches the documents it reaches to each input document. Use it for hierarchies and graphs — reporting chains, category trees, referral networks, or any relationship you would otherwise have to walk with repeated queries from the application.

## Syntax

```javascript
{
  $graphLookup: {
    from: <collection>,
    startWith: <expression>,
    connectFromField: <string>,
    connectToField: <string>,
    as: <string>,
    maxDepth: <nonNegativeInteger>,
    depthField: <string>,
    restrictSearchWithMatch: <document>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`from`** | Required. The collection to search recursively. |
| **`startWith`** | Required. An expression giving the value (or array of values) that begins the traversal. |
| **`connectFromField`** | Required. The field whose value is carried into the next round of the search. |
| **`connectToField`** | Required. The field in the `from` collection matched against the incoming value. |
| **`as`** | Required. Name of the array field added to each input document to hold the matched documents. Cannot begin with `$`. |
| **`maxDepth`** | Optional. A non-negative integer bounding how many recursive rounds run. Omit for an unbounded search. `0` visits only the documents matched by `startWith`. |
| **`depthField`** | Optional. Name of a field added to each matched document recording the recursion depth at which it was found, starting at `0`. Cannot begin with `$`. |
| **`restrictSearchWithMatch`** | Optional. A query document that matched documents must also satisfy. Documents that fail it are excluded, and the traversal does not continue through them. |

Any other parameter is rejected with `Unrecognized parameter supplied to stage $graphlookup`.

## Examples

The examples on this page use the following documents in an `employees` collection, each pointing at the person they report to by name.

```json
[
  { "_id": 1, "name": "Ana", "title": "VP, Retail", "reportsTo": null },
  { "_id": 2, "name": "Bo", "title": "Regional Manager", "reportsTo": "Ana" },
  { "_id": 3, "name": "Cai", "title": "Store Manager", "reportsTo": "Bo" },
  { "_id": 4, "name": "Dev", "title": "Shift Lead", "reportsTo": "Cai" },
  { "_id": 5, "name": "Eve", "title": "Associate", "reportsTo": "Dev" }
]
```

### Example 1: Walking a reporting chain

This query starts at Eve and follows `reportsTo` all the way to the top of the organization.

```javascript
db.employees.aggregate([
  { $match: { name: "Eve" } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$reportsTo",
      connectFromField: "reportsTo",
      connectToField: "name",
      as: "managementChain"
    }
  },
  { $project: { _id: 0, name: 1, "managementChain.name": 1 } }
])
```

This query returns the following result:

```json
[
  {
    "name": "Eve",
    "managementChain": [
      { "name": "Ana" },
      { "name": "Bo" },
      { "name": "Cai" },
      { "name": "Dev" }
    ]
  }
]
```

One pipeline reaches every ancestor. The order of the `as` array is not defined — use `depthField` when you need to know how far away each match was.

### Example 2: Bounding the search and recording depth

This query walks at most one extra level past the starting point and tags each match with the depth at which it was found.

```javascript
db.employees.aggregate([
  { $match: { name: "Eve" } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$reportsTo",
      connectFromField: "reportsTo",
      connectToField: "name",
      as: "managementChain",
      maxDepth: 1,
      depthField: "level"
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      "managementChain.name": 1,
      "managementChain.level": 1
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "name": "Eve",
    "managementChain": [
      { "name": "Cai", "level": 1 },
      { "name": "Dev", "level": 0 }
    ]
  }
]
```

Dev is Eve's direct manager, so it sits at depth `0`; Cai is one round further out at depth `1`. With `maxDepth: 1`, the search stops there.

### Example 3: Filtering which documents the search may traverse

This query follows the chain but only through people whose title contains `Lead` or `Manager`.

```javascript
db.employees.aggregate([
  { $match: { name: "Eve" } },
  {
    $graphLookup: {
      from: "employees",
      startWith: "$reportsTo",
      connectFromField: "reportsTo",
      connectToField: "name",
      as: "managementChain",
      restrictSearchWithMatch: { title: { $regex: "Lead|Manager" } }
    }
  },
  {
    $project: {
      _id: 0,
      name: 1,
      "managementChain.name": 1,
      "managementChain.title": 1
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "name": "Eve",
    "managementChain": [
      { "name": "Bo", "title": "Regional Manager" },
      { "name": "Cai", "title": "Store Manager" },
      { "name": "Dev", "title": "Shift Lead" }
    ]
  }
]
```

Ana is reachable but her title is `VP, Retail`, so she is filtered out — and because the traversal cannot pass through an excluded document, anyone above her would be unreachable as well.

## Error cases

| Problem | Error |
| --- | --- |
| `connectFromField` or `connectToField` missing | `Both 'connectFrom' and 'connectTo' operators must be provided for a $graphLookup operation.` |
| `from` missing | `$graphLookup requires 'from' field to be specified` |
| `as` missing | `$graphLookup requires 'as' field to be specified` |
| `startWith` missing | `You must provide a 'startWith' parameter when performing a $graphLookup operation` |
| Unknown parameter | `Unrecognized parameter supplied to stage $graphlookup: <name>` |
| Negative `maxDepth` | `The value of graphlookup.maxDepth must always be a non-negative integer number.` |

## Related

- [`$lookup`](../%24lookup/) — joins a single level instead of recursing.
- [`$unwind`](../%24unwind/) — flattens the array produced in the `as` field.
