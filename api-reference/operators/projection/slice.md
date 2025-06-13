# $slice

Projects a subset of elements from an array.

## Syntax

Single value (limit):
```javascript
{ <array>: { $slice: <number> } }
```

Skip and limit:
```javascript
{ <array>: { $slice: [ <skip>, <limit> ] } }
```

## Description

The `$slice` operator controls how many elements of an array to project. It can:
- Return the first or last N elements
- Skip elements and return a subset
- Work with nested array fields

## Examples

### Basic Array Slicing

```javascript
// Get first 3 elements
db.posts.find(
  {},
  {
    title: 1,
    comments: { $slice: 3 }
  }
)

// Get last 3 elements
db.posts.find(
  {},
  {
    title: 1,
    comments: { $slice: -3 }
  }
)
```

### Skip and Limit

```javascript
// Skip 2 elements and get next 3
db.articles.find(
  {},
  {
    title: 1,
    paragraphs: { $slice: [2, 3] }
  }
)
```

### Nested Arrays

```javascript
db.blogs.find(
  {},
  {
    title: 1,
    "sections.comments": { $slice: 5 },
    author: 1
  }
)
```

## Behavior

1. **Slice Operations**
   - Positive limit: First N elements
   - Negative limit: Last N elements
   - Skip and limit: Subset of elements
   - Preserves array order

2. **Array Handling**
   - Works with any array type
   - Preserves element structure
   - Handles nested arrays
   - Maintains data types

3. **Edge Cases**
   - Empty arrays
   - Limit exceeds array size
   - Skip exceeds array size
   - Non-array fields

## Use Cases

1. **Pagination**
   - Comment pagination
   - Result limiting
   - Page navigation
   - Data chunking

2. **Performance Optimization**
   - Reducing result size
   - Bandwidth optimization
   - Memory management
   - Load balancing

3. **Data Preview**
   - Recent items display
   - Content previews
   - Sample data
   - List truncation

## Best Practices

1. **Query Design**
   - Plan slice parameters
   - Consider array size
   - Handle edge cases
   - Optimize for use case

2. **Performance**
   - Use appropriate limits
   - Consider skip impact
   - Monitor memory usage
   - Optimize query patterns

3. **Data Handling**
   - Validate array fields
   - Handle empty results
   - Consider null values
   - Plan error cases

## Related Operators

- [$elemMatch](elemMatch.md) - Projects the first array element that matches the specified condition
- [$meta](meta.md) - Projects metadata from document queries 