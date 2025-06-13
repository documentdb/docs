# $addToSet

Adds elements to an array only if they do not already exist in the set.

## Syntax

```javascript
{ $addToSet: { <field>: <value> } }
```

With `$each` modifier:
```javascript
{ $addToSet: { <field>: { $each: [ <value1>, <value2>, ... ] } } }
```

## Description

The `$addToSet` operator adds a value to an array unless the value is already present, ensuring no duplicates are created. It treats the array as a set.

## Examples

### Basic Usage

```javascript
// Initial document
{ _id: 1, tags: ["red", "blue"] }

// Add a new tag
db.collection.update(
  { _id: 1 },
  { $addToSet: { tags: "green" } }
)

// Result
{ _id: 1, tags: ["red", "blue", "green"] }

// Try to add existing tag
db.collection.update(
  { _id: 1 },
  { $addToSet: { tags: "blue" } }
)

// Result (unchanged)
{ _id: 1, tags: ["red", "blue", "green"] }
```

### Using $each Modifier

```javascript
// Initial document
{ _id: 2, scores: [89, 90] }

// Add multiple unique scores
db.collection.update(
  { _id: 2 },
  {
    $addToSet: {
      scores: {
        $each: [85, 89, 92]
      }
    }
  }
)

// Result (89 not duplicated)
{ _id: 2, scores: [89, 90, 85, 92] }
```

### With Embedded Documents

```javascript
// Initial document
{
  _id: 3,
  users: [
    { id: 1, name: "John" },
    { id: 2, name: "Mary" }
  ]
}

// Add new unique user
db.collection.update(
  { _id: 3 },
  {
    $addToSet: {
      users: { id: 3, name: "Bob" }
    }
  }
)
```

## Behavior

1. **Value Comparison**
   - Exact value matching for primitives
   - Full document matching for objects
   - Order-sensitive for arrays

2. **Array Creation**
   - Creates array if field doesn't exist
   - Converts non-array field to array
   - Preserves existing array order

3. **Type Handling**
   - Maintains BSON type consistency
   - Treats different types as distinct
   - Respects array element ordering

## Use Cases

1. **Tag Management**
   - Adding unique tags to documents
   - Managing categories
   - Building feature sets

2. **User Collections**
   - Managing unique user lists
   - Building group memberships
   - Tracking unique interactions

3. **Data Aggregation**
   - Collecting unique values
   - Building distinct sets
   - Merging data sources

## Performance Considerations

1. **Array Size**
   - Monitor array growth
   - Consider document size limits
   - Use indexes for large arrays

2. **Operation Cost**
   - Linear search for duplicates
   - Consider array size impact
   - Use bulk operations when possible

3. **Index Usage**
   - Index array fields appropriately
   - Consider multikey index impact
   - Monitor index size

## Best Practices

1. **Document Design**
   - Plan for array growth
   - Consider embedding strategy
   - Monitor array size limits

2. **Operation Efficiency**
   - Use $each for multiple values
   - Batch related updates
   - Consider bulk operations

3. **Data Consistency**
   - Validate input data
   - Handle edge cases
   - Consider concurrency

## Related Operators

- [$push](push.md) - Adds an item to an array
- [$each](each.md) - Modifier for multiple values
- [$pull](pull.md) - Removes items from an array 