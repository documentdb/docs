# $slice

Limits the size of updated arrays.

## Syntax

```javascript
{
  $push: {
    <field>: {
      $each: [ <value1>, <value2>, ... ],
      $slice: <number>
    }
  }
}
```

## Description

The `$slice` operator modifier limits the number of array elements. It must be used with the `$push` operator and the `$each` modifier. When used with `$push`, it can maintain fixed-size arrays by trimming elements.

## Examples

### Limit Array Size

```javascript
// Initial document
{ _id: 1, scores: [85, 92, 78] }

// Add new score and keep only last 3
db.collection.update(
  { _id: 1 },
  {
    $push: {
      scores: {
        $each: [95],
        $slice: -3
      }
    }
  }
)

// Result
{ _id: 1, scores: [92, 78, 95] }
```

### Keep First N Elements

```javascript
// Initial document
{
  _id: 2,
  log: ["entry1", "entry2", "entry3", "entry4"]
}

// Add new entry and keep first 3
db.collection.update(
  { _id: 2 },
  {
    $push: {
      log: {
        $each: ["entry5"],
        $slice: 3
      }
    }
  }
)

// Result
{ _id: 2, log: ["entry1", "entry2", "entry3"] }
```

### Combine with Sort

```javascript
// Initial document
{
  _id: 3,
  scores: [
    { game: 1, points: 85 },
    { game: 2, points: 92 },
    { game: 3, points: 78 }
  ]
}

// Add new score, sort by points, keep top 3
db.collection.update(
  { _id: 3 },
  {
    $push: {
      scores: {
        $each: [{ game: 4, points: 95 }],
        $sort: { points: -1 },
        $slice: 3
      }
    }
  }
)

// Result (top 3 scores)
{
  _id: 3,
  scores: [
    { game: 4, points: 95 },
    { game: 2, points: 92 },
    { game: 1, points: 85 }
  ]
}
```

## Behavior

1. **Slice Values**
   - Negative: keep from end
   - Positive: keep from start
   - Zero: empty array

2. **Operation Order**
   - Applied after $sort
   - Applied after $each
   - Final array operation

3. **Array Requirements**
   - Must use with $push
   - Must use with $each
   - Creates array if needed

## Use Cases

1. **Fixed-Size Arrays**
   - Maintaining recent items
   - Limiting list size
   - Managing history

2. **Top-N Lists**
   - Keeping best scores
   - Recent activity logs
   - Performance rankings

3. **Data Management**
   - Capped collections
   - Rolling windows
   - Limited history

## Performance Considerations

1. **Memory Usage**
   - Efficient array trimming
   - No temporary copies
   - In-place modifications

2. **Operation Cost**
   - Single atomic operation
   - Combines with other modifiers
   - Efficient array management

3. **Index Impact**
   - Updates array indexes
   - Maintains index consistency
   - Consider index maintenance

## Best Practices

1. **Array Size**
   - Choose appropriate limits
   - Consider document size
   - Monitor array growth

2. **Operation Design**
   - Combine with $sort
   - Use with $push and $each
   - Plan array management

3. **Data Structure**
   - Consider access patterns
   - Plan for growth
   - Design for performance

## Common Patterns

### Recent Items List

```javascript
// Keep last 10 items
db.collection.update(
  { _id: 1 },
  {
    $push: {
      recentItems: {
        $each: [newItem],
        $slice: -10
      }
    }
  }
)
```

### Top Scores

```javascript
// Maintain top 5 scores
db.collection.update(
  { _id: 1 },
  {
    $push: {
      highScores: {
        $each: [newScore],
        $sort: { score: -1 },
        $slice: 5
      }
    }
  }
)
```

## Related Operators

- [$push](push.md) - Adds elements to an array
- [$each](each.md) - Adds multiple elements
- [$sort](sort.md) - Sorts array elements 