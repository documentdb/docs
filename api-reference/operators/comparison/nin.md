# $nin

Matches none of the values specified in an array.

## Syntax

```javascript
{ <field>: { $nin: [<value1>, <value2>, ... <valueN>] } }
```

## Description

The `$nin` operator selects documents where the field value is not in the specified array or the field does not exist. This operator is the inverse of the `$in` operator.

## Examples

### Basic Value Exclusion

```javascript
// Find documents where status is neither "archived" nor "deleted"
db.documents.find({
  status: {
    $nin: ["archived", "deleted"]
  }
})
```

### Multiple Types

```javascript
// Exclude documents with specific codes
db.items.find({
  code: {
    $nin: [123, "ABC", null]
  }
})
```

### Array Field Matching

```javascript
// Find documents where tags don't include specified values
db.posts.find({
  tags: {
    $nin: ["draft", "private", "unlisted"]
  }
})
```

## Behavior

1. **Type Handling**
   - Supports mixed types
   - Type-sensitive comparison
   - No type coercion

2. **Array Handling**
   - Matches non-matching arrays
   - Order doesn't matter
   - Supports nested arrays

3. **Special Cases**
   - Includes missing fields
   - Handles null values
   - Empty array handling

## Use Cases

1. **Status Filtering**
   ```javascript
   // Find active documents
   db.content.find({
     status: {
       $nin: ["deleted", "archived", "hidden"]
     }
   })
   ```

2. **Category Exclusion**
   ```javascript
   // Find products not in excluded categories
   db.products.find({
     category: {
       $nin: excludedCategories
     }
   })
   ```

3. **Role-based Access**
   ```javascript
   // Find users without specific roles
   db.users.find({
     roles: {
       $nin: ["banned", "suspended"]
     }
   })
   ```

## Performance Considerations

1. **Index Usage**
   - Limited index usage
   - Full collection scan possible
   - Consider alternatives

2. **Query Optimization**
   - Array size impact
   - Value distribution
   - Index selectivity

3. **Memory Usage**
   - Array size limits
   - Query plan caching
   - Result set size

## Best Practices

1. **Array Size**
   - Keep arrays reasonably sized
   - Consider chunking large arrays
   - Monitor performance

2. **Query Design**
   - Consider alternatives
   - Use with other operators
   - Check index usage

3. **Data Consistency**
   - Handle missing fields
   - Consider null values
   - Validate data types

## Common Patterns

### Exclusion Filtering

```javascript
// Find available products
db.products.find({
  status: {
    $nin: ["outOfStock", "discontinued"]
  }
})
```

### Access Control

```javascript
// Find accessible content
db.content.find({
  visibility: {
    $nin: ["private", "restricted"]
  }
})
```

### Error Handling

```javascript
// Find valid records
db.records.find({
  errorCode: {
    $nin: errorCodes
  }
})
```

## Related Operators

- [$in](in.md) - Matches any of the values in an array
- [$ne](ne.md) - Matches values that are not equal to a specified value
- [$not](../logical/not.md) - Inverts the effect of a query expression 