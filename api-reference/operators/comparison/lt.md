# $lt

Matches values that are less than a specified value.

## Syntax

```javascript
{ <field>: { $lt: <value> } }
```

## Description

The `$lt` operator selects documents where the value of the field is less than (i.e., <) the specified value. For comparisons between different BSON type values, refer to the BSON comparison order.

## Examples

### Basic Numeric Comparison

```javascript
// Find documents where price is less than 50
db.products.find({
  price: { $lt: 50 }
})
```

### Date Comparison

```javascript
// Find documents with dates before January 1, 2024
db.events.find({
  date: {
    $lt: ISODate("2024-01-01")
  }
})
```

### Nested Field Comparison

```javascript
// Find documents where nested value is below threshold
db.metrics.find({
  "stats.value": { $lt: maxThreshold }
})
```

## Behavior

1. **Type Handling**
   - Follows BSON comparison order
   - Type-sensitive comparison
   - No type coercion

2. **Special Values**
   - Handles null values
   - Handles missing fields
   - Respects data types

3. **Array Comparison**
   - Compares with array elements
   - Array order matters
   - Nested array handling

## Use Cases

1. **Price Filtering**
   ```javascript
   // Find budget-friendly items
   db.products.find({
     price: { $lt: maxPrice }
   })
   ```

2. **Date Range Queries**
   ```javascript
   // Find records before end date
   db.logs.find({
     timestamp: {
       $lt: endDate
     }
   })
   ```

3. **Performance Monitoring**
   ```javascript
   // Find instances below threshold
   db.metrics.find({
     responseTime: { $lt: threshold }
   })
   ```

## Performance Considerations

1. **Index Usage**
   - Can use indexes efficiently
   - Range query optimization
   - Compound index strategies

2. **Query Optimization**
   - Selective queries
   - Index bounds
   - Sort operations

3. **Memory Usage**
   - Efficient comparison
   - No regex overhead
   - Stream processing

## Best Practices

1. **Index Design**
   - Create appropriate indexes
   - Consider query patterns
   - Use compound indexes

2. **Data Types**
   - Maintain consistent types
   - Handle type conversion
   - Consider collation

3. **Query Structure**
   - Combine with other operators
   - Use with sort operations
   - Consider query selectivity

## Common Patterns

### Range Queries

```javascript
// Find items within price range
db.products.find({
  price: {
    $lt: maxPrice,
    $gt: minPrice
  }
})
```

### Resource Management

```javascript
// Find resources below capacity
db.servers.find({
  "resources.cpu": { $lt: warningThreshold },
  "resources.memory": { $lt: memoryLimit }
})
```

### Age Restrictions

```javascript
// Find minor users
db.users.find({
  age: { $lt: 18 }
})
```

## Related Operators

- [$lte](lte.md) - Matches values less than or equal to
- [$gt](gt.md) - Matches values greater than
- [$gte](gte.md) - Matches values greater than or equal to 