# Array Update Operators

Array update operators provide ways to modify array fields in documents. These operators allow you to add, remove, and modify elements in arrays.

## Update Operators

| Operator | Description |
|----------|-------------|
| [$addToSet](addToSet.md) | Adds elements to an array only if they do not already exist |
| [$pop](pop.md) | Removes the first or last item of an array |
| [$pull](pull.md) | Removes all array elements that match a specified query |
| [$push](push.md) | Adds an item to an array |
| [$pullAll](pullAll.md) | Removes all matching values from an array |

## Modifiers

| Operator | Description |
|----------|-------------|
| [$each](each.md) | Modifies $push and $addToSet to append multiple items |
| [$position](position.md) | Specifies the position in the array to add elements |
| [$slice](slice.md) | Limits the size of updated arrays |
| [$sort](sort.md) | Reorders documents stored in an array |

## Common Use Cases

- Adding elements to arrays
- Removing elements from arrays
- Updating array elements
- Managing array order and size
- Set-like operations on arrays

## Example Usage

```javascript
// Add unique elements to an array
db.collection.update(
  { _id: 1 },
  { $addToSet: { tags: "new-tag" } }
)

// Push multiple elements to an array
db.collection.update(
  { _id: 1 },
  {
    $push: {
      scores: {
        $each: [85, 92, 78],
        $sort: 1
      }
    }
  }
)

// Remove elements matching criteria
db.collection.update(
  { _id: 1 },
  { $pull: { scores: { $lt: 60 } } }
)
```

## Best Practices

1. **Array Growth**
   - Monitor array sizes to prevent document growth issues
   - Use $slice to limit array sizes when needed
   - Consider using $position for ordered insertions

2. **Performance**
   - Index array fields for better query performance
   - Use $pullAll instead of multiple $pull operations
   - Batch array updates when possible

3. **Data Consistency**
   - Use $addToSet for set-like behavior
   - Combine with array filters for precise updates
   - Consider atomic operations for critical updates 