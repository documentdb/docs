# $push

Adds an element to an array.

## Syntax

Basic syntax:
```javascript
{ $push: { <field>: <value> } }
```

With modifiers:
```javascript
{
  $push: {
    <field>: {
      $each: [ <value1>, <value2>, ... ],
      $slice: <number>,
      $sort: <sort specification>,
      $position: <number>
    }
  }
}
```

## Description

The `$push` operator appends a specified value to an array. When used with modifiers, it can perform complex array operations like adding multiple elements, maintaining array size, sorting, and inserting at specific positions.

## Examples

### Basic Push

```javascript
// Initial document
{ _id: 1, scores: [85, 92] }

// Add a single score
db.collection.update(
  { _id: 1 },
  { $push: { scores: 78 } }
)

// Result
{ _id: 1, scores: [85, 92, 78] }
```

### Push Multiple Elements

```javascript
// Initial document
{ _id: 2, tags: ["red", "blue"] }

// Add multiple tags
db.collection.update(
  { _id: 2 },
  {
    $push: {
      tags: {
        $each: ["green", "yellow"],
        $sort: 1
      }
    }
  }
)

// Result
{ _id: 2, tags: ["blue", "green", "red", "yellow"] }
```

### Complex Array Operations

```javascript
// Initial document
{
  _id: 3,
  scores: [
    { subject: "math", score: 85 },
    { subject: "english", score: 92 }
  ]
}

// Add scores with sort and slice
db.collection.update(
  { _id: 3 },
  {
    $push: {
      scores: {
        $each: [
          { subject: "science", score: 78 },
          { subject: "history", score: 88 }
        ],
        $sort: { score: -1 },
        $slice: 3
      }
    }
  }
)
```

## Modifiers

1. **$each**
   - Adds multiple elements
   - Maintains array order
   - Works with other modifiers

2. **$slice**
   - Limits array size
   - Negative values keep from end
   - Applied after other operations

3. **$sort**
   - Sorts array elements
   - Supports object sorting
   - Applied before slice

4. **$position**
   - Specifies insertion point
   - Zero-based index
   - Must use with $each

## Behavior

1. **Array Creation**
   - Creates array if not exists
   - Converts non-array to array
   - Preserves existing elements

2. **Type Handling**
   - Maintains BSON types
   - Supports mixed types
   - Preserves object structure

3. **Atomic Operation**
   - Single atomic update
   - All modifiers in one operation
   - Maintains consistency

## Use Cases

1. **Data Collection**
   - Adding new measurements
   - Recording events
   - Logging activities

2. **List Management**
   - Managing ordered lists
   - Maintaining top-N elements
   - Building sorted collections

3. **Queue Operations**
   - Implementing work queues
   - Managing message buffers
   - Building event logs

## Performance Considerations

1. **Array Growth**
   - Monitor array size
   - Use $slice for size control
   - Consider document limits

2. **Sort Operations**
   - Sort performance impact
   - Index usage with sort
   - Memory requirements

3. **Batch Operations**
   - Use $each for multiple items
   - Batch similar updates
   - Consider bulk operations

## Best Practices

1. **Array Management**
   - Control array growth
   - Use appropriate modifiers
   - Consider data structure

2. **Operation Efficiency**
   - Combine operations
   - Use bulk updates
   - Monitor performance

3. **Data Consistency**
   - Use atomic operations
   - Validate input data
   - Handle edge cases

## Common Patterns

### Capped Arrays

```javascript
// Maintain last 5 elements
db.collection.update(
  { _id: 1 },
  {
    $push: {
      logs: {
        $each: [newLog],
        $slice: -5
      }
    }
  }
)
```

### Sorted Lists

```javascript
// Add items and maintain sort
db.collection.update(
  { _id: 1 },
  {
    $push: {
      scores: {
        $each: [newScore],
        $sort: { value: -1 },
        $slice: 10
      }
    }
  }
)
```

## Related Operators

- [$addToSet](addToSet.md) - Adds unique elements
- [$each](each.md) - Adds multiple elements
- [$slice](slice.md) - Limits array size
- [$sort](sort.md) - Sorts array elements 