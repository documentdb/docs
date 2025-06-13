# $gt

Matches values that are greater than a specified value.

## Syntax

```javascript
{ <field>: { $gt: <value> } }
```

## Description

The `$gt` operator selects documents where the value of the field is greater than (i.e., >) the specified value. For comparisons between different BSON type values, refer to the BSON comparison order.

## Examples

### Basic Numeric Comparison

```javascript
// Find documents where price is greater than 100
db.products.find({
  price: { $gt: 100 }
})
```

### Date Comparison

```javascript
// Find documents with dates after January 1, 2024
db.events.find({
  date: {
    $gt: ISODate("2024-01-01")
  }
})
```

### Nested Field Comparison

```javascript
// Find documents where nested value is greater than threshold
db.metrics.find({
  "stats.count": { $gt: 1000 }
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

1. **Numeric Filtering**
   ```javascript
   // Find products above price threshold
   db.products.find({
     price: { $gt: minimumPrice }
   })
   ```

2. **Date Range Queries**
   ```javascript
   // Find records after specific date
   db.logs.find({
     timestamp: {
       $gt: startDate
     }
   })
   ```

3. **Version Comparison**
   ```javascript
   // Find newer versions
   db.software.find({
     version: { $gt: "2.0.0" }
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

### Price Filtering

```javascript
// Find premium products
db.products.find({
  price: { $gt: premiumThreshold }
})
```

### Age Restrictions

```javascript
// Find adult users
db.users.find({
  age: { $gt: 18 }
})
```

### Performance Metrics

```javascript
// Find high-performance instances
db.servers.find({
  "metrics.responseTime": { $gt: threshold }
})
```

## Related Operators

- [$gte](gte.md) - Matches values greater than or equal to
- [$lt](lt.md) - Matches values less than
- [$lte](lte.md) - Matches values less than or equal to 