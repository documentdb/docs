# $sort

Reorders array elements when used with array update operators.

## Syntax

For simple sorting:
```javascript
{
  $push: {
    <field>: {
      $each: [ <value1>, <value2>, ... ],
      $sort: <1 | -1>
    }
  }
}
```

For object sorting:
```javascript
{
  $push: {
    <field>: {
      $each: [ <value1>, <value2>, ... ],
      $sort: { <field1>: <1 | -1>, <field2>: <1 | -1>, ... }
    }
  }
}
```

## Description

The `$sort` operator modifier orders array elements when used with the `$push` operator and `$each` modifier. It can sort by scalar values or by fields within embedded documents.

## Examples

### Simple Sorting

```javascript
// Initial document
{ _id: 1, scores: [89, 70, 95, 82] }

// Add new score and sort ascending
db.collection.update(
  { _id: 1 },
  {
    $push: {
      scores: {
        $each: [77],
        $sort: 1
      }
    }
  }
)

// Result
{ _id: 1, scores: [70, 77, 82, 89, 95] }
```

### Object Sorting

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

// Add new grade and sort by score descending
db.collection.update(
  { _id: 2 },
  {
    $push: {
      grades: {
        $each: [{ subject: "history", score: 88 }],
        $sort: { score: -1 }
      }
    }
  }
)

// Result
{
  _id: 2,
  grades: [
    { subject: "english", score: 92 },
    { subject: "history", score: 88 },
    { subject: "math", score: 85 },
    { subject: "science", score: 78 }
  ]
}
```

### Multiple Sort Fields

```javascript
// Initial document
{
  _id: 3,
  items: [
    { type: "A", priority: 1, value: 10 },
    { type: "B", priority: 2, value: 15 },
    { type: "A", priority: 2, value: 20 }
  ]
}

// Sort by multiple fields
db.collection.update(
  { _id: 3 },
  {
    $push: {
      items: {
        $each: [{ type: "B", priority: 1, value: 25 }],
        $sort: { priority: -1, type: 1 }
      }
    }
  }
)
```

## Behavior

1. **Sort Order**
   - 1: ascending order
   - -1: descending order
   - Multiple fields supported

2. **Operation Order**
   - Applied before $slice
   - Applied after $each
   - Sorts entire array

3. **Type Handling**
   - Follows BSON type order
   - Mixed types sorted by type
   - Null values handled

## Use Cases

1. **Data Organization**
   - Maintaining sorted lists
   - Priority queues
   - Ranked collections

2. **Display Order**
   - User preferences
   - Natural ordering
   - Custom sorting

3. **Data Analysis**
   - Sorted statistics
   - Performance rankings
   - Time-based ordering

## Performance Considerations

1. **Array Size**
   - Sort complexity O(n log n)
   - Consider array length
   - Use with $slice

2. **Memory Usage**
   - Temporary sort space
   - Document size impact
   - Index maintenance

3. **Operation Cost**
   - Single atomic operation
   - Combined with other modifiers
   - Server-side sorting

## Best Practices

1. **Sort Design**
   - Choose appropriate fields
   - Consider data types
   - Plan sort order

2. **Operation Efficiency**
   - Combine with $slice
   - Use selective sorting
   - Monitor performance

3. **Data Structure**
   - Plan field selection
   - Consider index support
   - Optimize for common sorts

## Common Patterns

### Top-N Pattern

```javascript
// Maintain top 5 scores
db.collection.update(
  { _id: 1 },
  {
    $push: {
      scores: {
        $each: [newScore],
        $sort: -1,
        $slice: 5
      }
    }
  }
)
```

### Priority Queue

```javascript
// Sort by priority and timestamp
db.collection.update(
  { _id: 1 },
  {
    $push: {
      queue: {
        $each: [newItem],
        $sort: {
          priority: -1,
          timestamp: 1
        }
      }
    }
  }
)
```

## Related Operators

- [$push](push.md) - Adds elements to an array
- [$each](each.md) - Adds multiple elements
- [$slice](slice.md) - Limits array size 