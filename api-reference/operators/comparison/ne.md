# $ne

Matches values that are not equal to a specified value.

## Syntax

```javascript
{ <field>: { $ne: <value> } }
```

## Description

The `$ne` operator selects documents where the value of the field is not equal to (i.e., !=) the specified value. This includes documents that don't contain the field. For comparisons between different BSON type values, refer to the BSON comparison order.

## Examples

### Basic Value Comparison

```javascript
// Find documents where status is not "completed"
db.tasks.find({
  status: { $ne: "completed" }
})
```

### Numeric Comparison

```javascript
// Find documents where quantity is not zero
db.inventory.find({
  quantity: { $ne: 0 }
})
```

### Nested Field Comparison

```javascript
// Find documents where nested value is not default
db.settings.find({
  "config.mode": { $ne: "default" }
})
```

## Behavior

1. **Type Handling**
   - Type-sensitive comparison
   - No type coercion
   - Follows BSON comparison order

2. **Special Values**
   - Includes missing fields
   - Handles null values
   - Array element matching

3. **Array Comparison**
   - Matches non-matching arrays
   - Element-wise comparison
   - Order-sensitive

## Use Cases

1. **Exclusion Filtering**
   ```javascript
   // Find active items
   db.items.find({
     status: { $ne: "deleted" }
   })
   ```

2. **Error Detection**
   ```javascript
   // Find non-zero error counts
   db.logs.find({
     errorCount: { $ne: 0 }
   })
   ```

3. **Configuration Validation**
   ```javascript
   // Find non-default configurations
   db.settings.find({
     mode: { $ne: "default" }
   })
   ```

## Performance Considerations

1. **Index Usage**
   - Limited index usage
   - Full collection scan possible
   - Consider alternatives

2. **Query Optimization**
   - Less selective
   - High cardinality impact
   - Compound index limitations

3. **Memory Usage**
   - Result set size
   - Query plan caching
   - Memory constraints

## Best Practices

1. **Query Design**
   - Consider $nin for multiple values
   - Use with other operators
   - Check index usage

2. **Performance**
   - Monitor query performance
   - Consider alternatives
   - Use appropriate indexes

3. **Data Consistency**
   - Handle missing fields
   - Consider null values
   - Validate data types

## Common Patterns

### Status Filtering

```javascript
// Find non-archived items
db.documents.find({
  status: { $ne: "archived" }
})
```

### Error Checking

```javascript
// Find records with non-zero errors
db.logs.find({
  errors: { $ne: 0 },
  status: "processed"
})
```

### Configuration Management

```javascript
// Find custom configurations
db.settings.find({
  "config.type": { $ne: "standard" }
})
```

## Related Operators

- [$eq](eq.md) - Matches values equal to a specified value
- [$nin](nin.md) - Matches none of the values in an array
- [$not](../logical/not.md) - Inverts the effect of a query expression 