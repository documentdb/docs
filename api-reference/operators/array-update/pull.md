# $pull

Removes all array elements that match a specified query.

## Syntax

```javascript
{ $pull: { <field>: <value|condition> } }
```

## Description

The `$pull` operator removes from an existing array all instances of a value or values that match a specified condition.

## Examples

### Remove Specific Values

```javascript
// Initial document
{ _id: 1, scores: [85, 92, 78, 85, 92] }

// Remove all instances of 85
db.collection.update(
  { _id: 1 },
  { $pull: { scores: 85 } }
)

// Result
{ _id: 1, scores: [92, 78, 92] }
```

### Remove Elements Matching a Condition

```javascript
// Initial document
{
  _id: 2,
  grades: [
    { subject: "math", score: 85 },
    { subject: "english", score: 92 },
    { subject: "science", score: 78 }
  ]
}

// Remove grades below 80
db.collection.update(
  { _id: 2 },
  {
    $pull: {
      grades: { score: { $lt: 80 } }
    }
  }
)

// Result
{
  _id: 2,
  grades: [
    { subject: "math", score: 85 },
    { subject: "english", score: 92 }
  ]
}
```

### Multiple Conditions

```javascript
// Initial document
{
  _id: 3,
  items: [
    { type: "food", qty: 5, price: 10 },
    { type: "drink", qty: 2, price: 5 },
    { type: "food", qty: 1, price: 15 }
  ]
}

// Remove food items with qty > 2
db.collection.update(
  { _id: 3 },
  {
    $pull: {
      items: {
        type: "food",
        qty: { $gt: 2 }
      }
    }
  }
)
```

## Behavior

1. **Matching**
   - Removes all matching elements
   - Supports query operators
   - Exact match for objects

2. **Array Requirements**
   - Must operate on arrays
   - Error on non-array fields
   - No effect on empty arrays

3. **Query Conditions**
   - Supports comparison operators
   - Can use logical operators
   - Matches array elements

## Use Cases

1. **Data Cleanup**
   - Removing invalid entries
   - Filtering unwanted data
   - Maintaining data quality

2. **List Management**
   - Removing obsolete items
   - Filtering by criteria
   - Managing collections

3. **Data Filtering**
   - Removing outliers
   - Filtering by conditions
   - Cleaning datasets

## Performance Considerations

1. **Array Scanning**
   - Full array scan required
   - Linear time complexity
   - Consider array size

2. **Index Usage**
   - Indexes don't optimize $pull
   - Plan for full array scan
   - Monitor performance

3. **Document Updates**
   - Single atomic operation
   - Updates whole document
   - Consider document size

## Best Practices

1. **Query Design**
   - Use specific conditions
   - Combine multiple criteria
   - Consider alternatives

2. **Error Handling**
   - Validate array field
   - Check conditions
   - Handle edge cases

3. **Operation Batching**
   - Batch similar operations
   - Use bulk updates
   - Consider transactions

## Common Patterns

### Removing Duplicates

```javascript
// Remove duplicate values
db.collection.update(
  { _id: 1 },
  {
    $pull: {
      items: {
        $in: db.collection.distinct("items", { _id: 1 })
      }
    }
  }
)
```

### Conditional Removal

```javascript
// Remove based on multiple conditions
db.collection.update(
  { _id: 1 },
  {
    $pull: {
      scores: {
        $or: [
          { value: { $lt: 60 } },
          { status: "invalid" }
        ]
      }
    }
  }
)
```

## Related Operators

- [$pullAll](pullAll.md) - Removes all matching values
- [$pop](pop.md) - Removes first or last element
- [$push](push.md) - Adds elements to an array 