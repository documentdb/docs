# $pop

Removes the first or last element of an array.

## Syntax

```javascript
{ $pop: { <field>: <-1 | 1> } }
```

Where:
- `-1` removes the first element of the array
- `1` removes the last element of the array

## Description

The `$pop` operator removes either the first or last element of an array. It provides a way to use arrays as queues or stacks.

## Examples

### Remove Last Element

```javascript
// Initial document
{ _id: 1, scores: [85, 92, 78] }

// Remove last element
db.collection.update(
  { _id: 1 },
  { $pop: { scores: 1 } }
)

// Result
{ _id: 1, scores: [85, 92] }
```

### Remove First Element

```javascript
// Initial document
{ _id: 2, queue: ["first", "second", "third"] }

// Remove first element
db.collection.update(
  { _id: 2 },
  { $pop: { queue: -1 } }
)

// Result
{ _id: 2, queue: ["second", "third"] }
```

### Multiple Array Fields

```javascript
// Initial document
{
  _id: 3,
  stack1: [1, 2, 3],
  stack2: [4, 5, 6]
}

// Pop from multiple arrays
db.collection.update(
  { _id: 3 },
  {
    $pop: {
      stack1: 1,
      stack2: 1
    }
  }
)

// Result
{
  _id: 3,
  stack1: [1, 2],
  stack2: [4, 5]
}
```

## Behavior

1. **Empty Arrays**
   - No effect on empty arrays
   - Operation succeeds silently
   - Maintains array field

2. **Non-Array Fields**
   - Error on non-array fields
   - Field must be an array
   - No automatic conversion

3. **Value Requirements**
   - Only accepts 1 or -1
   - Other values cause error
   - No effect if invalid value

## Use Cases

1. **Queue Management**
   - Implementing FIFO queues
   - Managing work items
   - Processing sequences

2. **Stack Operations**
   - Implementing LIFO stacks
   - Managing undo operations
   - Tracking history

3. **List Maintenance**
   - Trimming lists
   - Managing fixed-size arrays
   - Removing outdated items

## Performance Considerations

1. **Operation Speed**
   - Constant time operation
   - No array scanning needed
   - Efficient for large arrays

2. **Document Updates**
   - Single atomic operation
   - Minimal reallocation
   - Efficient space usage

3. **Index Impact**
   - Updates array indexes
   - Maintains index consistency
   - Minimal index overhead

## Best Practices

1. **Error Handling**
   - Check array existence
   - Validate array state
   - Handle edge cases

2. **Atomic Operations**
   - Use in transactions
   - Consider concurrency
   - Maintain consistency

3. **Data Validation**
   - Verify array bounds
   - Check array content
   - Monitor array size

## Common Patterns

### Queue Processing

```javascript
// Process queue items FIFO
db.collection.update(
  { queue: { $exists: true, $ne: [] } },
  { $pop: { queue: -1 } }
)
```

### Stack Management

```javascript
// Implement undo stack
db.collection.update(
  { history: { $exists: true, $ne: [] } },
  { $pop: { history: 1 } }
)
```

## Related Operators

- [$pull](pull.md) - Removes items matching a condition
- [$pullAll](pullAll.md) - Removes multiple items
- [$push](push.md) - Adds items to an array 