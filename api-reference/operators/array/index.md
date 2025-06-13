# Array Query Operators

Array query operators provide ways to query and match array fields in documents. These operators allow you to search for specific elements, match conditions within arrays, and query based on array properties.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$all](all.md) | Matches arrays that contain all specified elements |
| [$elemMatch](elemMatch.md) | Matches documents where array elements meet all specified conditions |
| [$size](size.md) | Matches arrays with a specific number of elements |

## Common Use Cases

- Finding documents with specific array elements
- Matching arrays containing all required values
- Filtering based on array length
- Complex array element condition matching

## Example Usage

```javascript
// Find documents where tags array contains all specified elements
db.products.find({ tags: { $all: ["electronics", "mobile"] } })

// Find documents where at least one review meets all criteria
db.products.find({
  reviews: {
    $elemMatch: {
      rating: { $gte: 4 },
      verified: true
    }
  }
})

// Find documents with exactly 3 comments
db.posts.find({ comments: { $size: 3 } })
```

## Best Practices

1. **Index Usage**
   - Create indexes on array fields for better query performance
   - Consider multikey indexes for array fields

2. **Query Optimization**
   - Use `$elemMatch` for complex conditions on array elements
   - Combine array operators with other query operators for precise filtering

3. **Performance**
   - Be mindful of array size when using array operators
   - Consider the impact of array scanning on query performance 