# $bottom

Returns the bottom element within a group according to the specified sort order.

## Syntax

```javascript
{
  $bottom: {
    sortBy: { <field1>: <sort order>, ... },
    output: <expression>
  }
}
```

## Description

The `$bottom` accumulator returns the element with the lowest value according to the specified sort criteria. It requires:
- `sortBy`: Specifies the fields and their sort order (1 for ascending, -1 for descending)
- `output`: Optional. Specifies the value to return. If omitted, returns the entire document

## Examples

### Basic Usage

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      lowestSale: {
        $bottom: {
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
      lowestScorer: {
        $bottom: {
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
        lowestPrice: {
          $bottom: {
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

- Returns `null` if the group is empty
- If multiple documents have the same lowest value, returns the first one encountered
- Must specify at least one field in `sortBy`
- Can be used in `$group`, `$bucket`, `$bucketAuto`, and `$setWindowFields` stages

## Use Cases

- Finding the lowest value record in a group
- Identifying minimum transactions
- Getting the earliest or latest record based on multiple criteria
- Finding extreme values in time series data

## Related Operators

- [$bottomN](bottomN.md) - Returns an array of the bottom N elements
- [$top](top.md) - Returns the top element
- [$topN](topN.md) - Returns an array of the top N elements 