# $top

Returns the top element within a group according to the specified sort order.

## Syntax

```javascript
{
  $top: {
    sortBy: { <field1>: <sort order>, ... },
    output: <expression>
  }
}
```

## Description

The `$top` accumulator returns the element with the highest value according to the specified sort criteria. It requires:
- `sortBy`: Specifies the fields and their sort order (1 for ascending, -1 for descending)
- `output`: Optional. Specifies the value to return. If omitted, returns the entire document

## Examples

### Basic Usage

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      highestSale: {
        $top: {
          sortBy: { amount: -1 },
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
      topScorer: {
        $top: {
          sortBy: { 
            score: -1,
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
        highestPrice: {
          $top: {
            sortBy: { price: -1 },
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

- Returns `null` if the group is empty
- If multiple documents have the same highest value, returns the first one encountered
- Must specify at least one field in `sortBy`
- Can be used in `$group`, `$bucket`, `$bucketAuto`, and `$setWindowFields` stages

## Use Cases

- Finding the highest value record in a group
- Identifying maximum transactions
- Getting the earliest or latest record based on multiple criteria
- Finding extreme values in time series data
- Determining winners or top performers

## Related Operators

- [$topN](topN.md) - Returns an array of the top N elements
- [$bottom](bottom.md) - Returns the bottom element
- [$bottomN](bottomN.md) - Returns an array of the bottom N elements 