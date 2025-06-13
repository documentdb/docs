# $bottomN

Returns an array of the bottom N elements within a group according to the specified sort order.

## Syntax

```javascript
{
  $bottomN: {
    n: <positive integer>,
    sortBy: { <field1>: <sort order>, ... },
    output: <expression>
  }
}
```

## Description

The `$bottomN` accumulator returns an array of the N elements with the lowest values according to the specified sort criteria. It requires:
- `n`: The number of elements to return (must be positive)
- `sortBy`: Specifies the fields and their sort order (1 for ascending, -1 for descending)
- `output`: Optional. Specifies the value to return for each element. If omitted, returns entire documents

## Examples

### Basic Usage

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      bottomThreeSales: {
        $bottomN: {
          n: 3,
          sortBy: { amount: 1 },
          output: {
            amount: "$amount",
            date: "$date",
            customer: "$customer"
          }
        }
      }
    }
  }
])
```

### Multiple Sort Fields

```javascript
db.students.aggregate([
  {
    $group: {
      _id: "$class",
      lowestFiveScorers: {
        $bottomN: {
          n: 5,
          sortBy: { 
            score: 1,
            lastName: 1 
          }
        }
      }
    }
  }
])
```

### With Window Functions

```javascript
db.stocks.aggregate([
  {
    $setWindowFields: {
      partitionBy: "$symbol",
      sortBy: { date: 1 },
      output: {
        lowestThreePrices: {
          $bottomN: {
            n: 3,
            sortBy: { price: 1 },
            output: "$price"
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

## Behavior

- Returns an empty array if the group is empty
- Returns all elements if N is greater than the number of elements in the group
- Elements are returned in ascending order according to the sort criteria
- Must specify at least one field in `sortBy`
- Can be used in `$group`, `$bucket`, `$bucketAuto`, and `$setWindowFields` stages

## Use Cases

- Finding the N lowest value records in a group
- Identifying bottom performers
- Getting the N earliest or latest records based on multiple criteria
- Analyzing extreme values in time series data
- Creating ranked lists or leaderboards (in reverse order)

## Related Operators

- [$bottom](bottom.md) - Returns the bottom element
- [$top](top.md) - Returns the top element
- [$topN](topN.md) - Returns an array of the top N elements 