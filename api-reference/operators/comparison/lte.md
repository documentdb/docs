# $lte

Matches values that are less than or equal to a specified value.

## Syntax

```javascript
{ <field>: { $lte: <value> } }
```

## Description

The `$lte` operator selects documents where the value of the field is less than or equal to (i.e., <=) the specified value. For comparisons between different BSON type values, refer to the BSON comparison order.

## Examples

### Basic Numeric Comparison

```javascript
// Find documents where quantity is less than or equal to 100
db.inventory.find({
  quantity: { $lte: 100 }
})
```

### Date Comparison

```javascript
// Find documents with dates up to January 1, 2024
db.events.find({
  date: {
    $lte: ISODate("2024-01-01")
  }
})
```

### Nested Field Comparison

```javascript
// Find documents where nested value is at most maximum threshold
db.metrics.find({
  "stats.value": { $lte: maxThreshold }
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

1. **Inventory Control**
   ```javascript
   // Find products needing restock
   db.products.find({
     stockLevel: { $lte: reorderPoint }
   })
   ```

2. **Date Range Queries**
   ```javascript
   // Find records up to end date
   db.transactions.find({
     timestamp: {
       $lte: endDate
     }
   })
   ```

3. **Score Filtering**
   ```javascript
   // Find failing grades
   db.grades.find({
     score: { $lte: passingThreshold }
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
    $gte: minPrice,
    $lte: maxPrice
  }
})
```

### Resource Limits

```javascript
// Find resources within limits
db.servers.find({
  "resources.usage": { $lte: maxCapacity }
})
```

### Version Compatibility

```javascript
// Find compatible versions
db.software.find({
  version: { $lte: maxVersion }
})
```

## Related Operators

- [$lt](lt.md) - Matches values less than
- [$gt](gt.md) - Matches values greater than
- [$gte](gte.md) - Matches values greater than or equal to 