# $last

Returns the last value in a group of documents or an array.

## Syntax

```javascript
{ $last: <expression> }
```

When used in `$setWindowFields`:
```javascript
{
  $last: {
    sortBy: { <field>: <1|-1>, ... },
    output: <expression>
  }
}
```

## Description

The `$last` accumulator returns:
- In a `$group` stage: The last document's value for the specified field in each group
- In a `$setWindowFields` stage: The last value in the window according to the sort order
- In a `$project` or `$addFields` stage: The last element of an array

## Examples

### Basic Group Example

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: "$item",
      lastSaleDate: { $last: "$date" }
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
        lastPrice: {
          $last: "$price",
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
      lastTag: { $last: "$tags" }
    }
  }
])
```

## Behavior

- Returns `null` if the group or array is empty
- When used in `$group`, the order of documents is not guaranteed unless preceded by a `$sort` stage
- In `$setWindowFields`, the sort order is determined by the `sortBy` parameter
- For arrays, returns the last element in the array's original order

## Use Cases

- Getting the most recent record in a time series
- Finding the last element in grouped data
- Extracting the last element from arrays
- Analyzing final states or outcomes
- Tracking most recent changes

## Related Operators

- [$first](first.md) - Returns the first value in a group
- [$firstN](firstN.md) - Returns the first N values in a group
- [$lastN](lastN.md) - Returns the last N values in a group 