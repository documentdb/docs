# $firstN

Returns an array of the first N values in a group of documents or an array.

## Syntax

```javascript
{ $firstN: { n: <positive integer>, input: <expression> } }
```

When used in `$setWindowFields`:
```javascript
{
  $firstN: {
    n: <positive integer>,
    input: <expression>,
    sortBy: { <field>: <1|-1>, ... }
  }
}
```

## Description

The `$firstN` accumulator returns:
- In a `$group` stage: The first N values for the specified field in each group
- In a `$setWindowFields` stage: The first N values in the window according to the sort order
- In a `$project` or `$addFields` stage: The first N elements of an array

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
      firstThreeSales: {
        $firstN: {
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
        firstThreePrices: {
          $firstN: {
            n: 3,
            input: "$price",
            sortBy: { date: 1 }
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
      firstThreeTags: {
        $firstN: {
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

## Use Cases

- Getting the first N records in a time series
- Sampling initial elements from grouped data
- Extracting the first N elements from arrays
- Creating limited previews of data
- Building top-N lists based on document order

## Related Operators

- [$first](first.md) - Returns the first value in a group
- [$last](last.md) - Returns the last value in a group
- [$lastN](lastN.md) - Returns the last N values in a group 