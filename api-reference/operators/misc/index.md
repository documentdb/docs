# Miscellaneous Operators

Miscellaneous operators provide utility functionality that doesn't fit into other operator categories.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$comment](comment.md) | Adds a comment to a query expression |
| [$rand](rand.md) | Generates a random float between 0 and 1 |

## Common Use Cases

- Adding documentation to complex queries
- Debugging and logging
- Random sampling and selection
- Testing and simulation

## Example Usage

```javascript
// Adding a comment to a query
db.collection.find({
  $comment: "This query finds active users",
  status: "active"
})

// Using random values for sampling
db.collection.aggregate([
  {
    $match: {
      $expr: {
        $lt: [ { $rand: {} }, 0.1 ] // 10% random sample
      }
    }
  }
])
```

## Best Practices

1. **Documentation**
   - Use meaningful comments
   - Document complex queries
   - Include relevant context

2. **Random Operations**
   - Use appropriate seed values
   - Consider distribution requirements
   - Handle edge cases

3. **Performance**
   - Be aware of comment overhead
   - Use random operations judiciously
   - Consider query optimization 