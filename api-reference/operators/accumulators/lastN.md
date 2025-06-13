# $lastN

Returns an array of the last N values in a group of documents or an array.

## Syntax

```javascript
{ $lastN: { n: <positive integer>, input: <expression> } }
```

When used in `$setWindowFields`:
```javascript
{
  $lastN: {
    n: <positive integer>,
    input: <expression>,
    sortBy: { <field>: <1|-1>, ... }
  }
}
```

## Description

The `$lastN` accumulator returns:
- In a `$group` stage: The last N values for the specified field in each group
- In a `$setWindowFields` stage: The last N values in the window according to the sort order
- In a `$project` or `$addFields` stage: The last N elements of an array

Required parameters:
- `n`: The number of elements to return (must be positive)
- `input`: The expression to evaluate and accumulate

Optional parameters:
- `sortBy`: (Only in `$setWindowFields`) Specifies the sort order for window operations

## Examples

### Basic Group Example

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      lastThreeSales: {
        $lastN: {
          n: 3,
          input: {
            date: "$date",
            amount: "$amount"
          }
        }
      }
    }
  }
])
```

### Window Function Example

```javascript
db.stocks.aggregate([
  {
    $setWindowFields: {
      partitionBy: "$symbol",
      sortBy: { date: 1 },
      output: {
        lastThreePrices: {
          $lastN: {
            n: 3,
            input: "$price",
            sortBy: { date: -1 }
          },
          window: {
            documents: ["unbounded", "current"]
          }
        }
      }
    }
  }
])
```

### Array Example

```javascript
db.inventory.aggregate([
  {
    $project: {
      item: 1,
      lastThreeTags: {
        $lastN: {
          n: 3,
          input: "$tags"
        }
      }
    }
  }
])
```

## Behavior

- Returns an empty array if the group or input array is empty
- Returns all elements if N is greater than the number of available elements
- When used in `$group`, the order of documents is not guaranteed unless preceded by a `$sort` stage
- In `$setWindowFields`, the sort order is determined by the `sortBy` parameter
- N must be a positive integer
- Elements are returned in their original order (not reversed)

## Use Cases

- Getting the most recent N records in a time series
- Analyzing recent trends or patterns
- Creating rolling windows of latest data
- Building history or audit trails
- Implementing "last N" caches or buffers

## Related Operators

- [$last](last.md) - Returns the last value in a group
- [$first](first.md) - Returns the first value in a group
- [$firstN](firstN.md) - Returns the first N values in a group 