# Projection Operators

Projection operators control how documents are transformed and which fields are included in the query results.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$elemMatch](elemMatch.md) | Projects the first array element that matches the specified condition |
| [$meta](meta.md) | Projects metadata from document queries |
| [$slice](slice.md) | Projects a subset of elements from an array |

## Common Use Cases

- Filtering array elements
- Accessing document metadata
- Limiting array results
- Optimizing query results
- Reducing document size

## Example Usage

```javascript
// Project matching array elements
db.collection.find(
  { tags: { $exists: true } },
  {
    name: 1,
    matchingTags: {
      $elemMatch: { $in: ["important", "urgent"] }
    }
  }
)

// Project document metadata
db.collection.find(
  { $text: { $search: "example" } },
  {
    title: 1,
    score: { $meta: "textScore" }
  }
)

// Project array subset
db.collection.find(
  { },
  {
    title: 1,
    recentComments: { $slice: ["$comments", -5] }  // Last 5 comments
  }
)
```

## Best Practices

1. **Field Selection**
   - Include only needed fields
   - Consider query performance
   - Optimize for data transfer

2. **Array Handling**
   - Use appropriate array limits
   - Consider ordering requirements
   - Handle empty arrays

3. **Performance**
   - Minimize projected fields
   - Use covered queries when possible
   - Consider index coverage 