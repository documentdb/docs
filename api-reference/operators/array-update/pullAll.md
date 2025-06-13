# $pullAll

Removes all matching values from an array.

## Syntax

```javascript
{ $pullAll: { <field>: [ <value1>, <value2>, ... ] } }
```

## Description

The `$pullAll` operator removes all instances of the specified values from an existing array. Unlike `$pull`, which can use query operators, `$pullAll` performs only exact matches.

## Examples

### Remove Multiple Values

```javascript
// Initial document
{ _id: 1, scores: [85, 92, 78, 85, 92, 78] }

// Remove all instances of 85 and 92
db.collection.update(
  { _id: 1 },
  { $pullAll: { scores: [85, 92] } }
)

// Result
{ _id: 1, scores: [78, 78] }
```

### Remove Objects

```javascript
// Initial document
{
  _id: 2,
  items: [
    { type: "A", qty: 5 },
    { type: "B", qty: 7 },
    { type: "A", qty: 5 }
  ]
}

// Remove all matching objects
db.collection.update(
  { _id: 2 },
  {
    $pullAll: {
      items: [
        { type: "A", qty: 5 }
      ]
    }
  }
)

// Result
{
  _id: 2,
  items: [
    { type: "B", qty: 7 }
  ]
}
```

### Multiple Array Fields

```javascript
// Initial document
{
  _id: 3,
  list1: [1, 2, 3, 1, 2],
  list2: ["a", "b", "a"]
}

// Remove from multiple arrays
db.collection.update(
  { _id: 3 },
  {
    $pullAll: {
      list1: [1, 2],
      list2: ["a"]
    }
  }
)

// Result
{
  _id: 3,
  list1: [3],
  list2: ["b"]
}
```

## Behavior

1. **Matching**
   - Exact value matching only
   - No query operators supported
   - Case-sensitive matching

2. **Array Requirements**
   - Must operate on arrays
   - Error on non-array fields
   - No effect on empty arrays

3. **Value Comparison**
   - Type-sensitive matching
   - Full object matching
   - Order-sensitive for arrays

## Use Cases

1. **Data Cleanup**
   - Removing specific values
   - Cleaning duplicate data
   - Batch value removal

2. **List Management**
   - Removing known values
   - Batch element removal
   - Set operations

3. **Data Normalization**
   - Removing unwanted values
   - Standardizing arrays
   - Cleaning datasets

## Performance Considerations

1. **Operation Speed**
   - Single atomic operation
   - More efficient than multiple $pull
   - Linear scan required

2. **Memory Usage**
   - Minimal memory overhead
   - No temporary arrays
   - Efficient value matching

3. **Document Updates**
   - Updates whole document
   - Consider document size
   - Plan for write load

## Best Practices

1. **Operation Design**
   - Group removals together
   - Use for known values
   - Consider alternatives

2. **Error Handling**
   - Validate array field
   - Check value types
   - Handle edge cases

3. **Batch Operations**
   - Combine related updates
   - Use bulk operations
   - Monitor performance

## Common Patterns

### Cleanup Operations

```javascript
// Remove multiple invalid values
db.collection.update(
  { _id: 1 },
  {
    $pullAll: {
      values: [null, "", undefined]
    }
  }
)
```

### Batch Removal

```javascript
// Remove multiple categories
db.collection.update(
  { _id: 1 },
  {
    $pullAll: {
      categories: ["deprecated", "obsolete", "legacy"]
    }
  }
)
```

## Comparison with $pull

1. **$pullAll**
   - Exact matches only
   - Multiple values at once
   - No query operators
   - More efficient for known values

2. **$pull**
   - Supports query conditions
   - Single condition at a time
   - More flexible matching
   - Better for complex conditions

## Related Operators

- [$pull](pull.md) - Removes elements matching a condition
- [$pop](pop.md) - Removes first or last element
- [$push](push.md) - Adds elements to an array 