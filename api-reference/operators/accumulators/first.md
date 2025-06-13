# $first

Returns the first value in a group of documents or an array.

## Syntax

```javascript
{ $first: <expression> }
```

When used in `$setWindowFields`:
```javascript
{
  $first: {
    sortBy: { <field>: <1|-1>, ... },
    output: <expression>
  }
}
```

## Description

The `$first` accumulator returns:
- In a `$group` stage: The first document's value for the specified field in each group
- In a `$setWindowFields` stage: The first value in the window according to the sort order
- In a `$project` or `$addFields` stage: The first element of an array

## Examples

### Basic Group Example

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      firstSaleDate: { $first: "$date" }
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
        firstPrice: {
          $first: "$price",
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
      firstTag: { $first: "$tags" }
    }
  }
])
```

## Behavior

- Returns `null` if the group or array is empty
- When used in `$group`, the order of documents is not guaranteed unless preceded by a `$sort` stage
- In `$setWindowFields`, the sort order is determined by the `sortBy` parameter

## Use Cases

- Getting the earliest record in a time series
- Finding the first element in grouped data
- Extracting the first element from arrays

## Related Operators

- [$last](last.md) - Returns the last value in a group
- [$firstN](firstN.md) - Returns the first N values in a group
- [$lastN](lastN.md) - Returns the last N values in a group 