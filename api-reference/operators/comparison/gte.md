# $gte

Matches values that are greater than or equal to a specified value.

## Syntax

```javascript
{ <field>: { $gte: <value> } }
```

## Description

The `$gte` operator selects documents where the value of the field is greater than or equal to (i.e., >=) the specified value. For comparisons between different BSON type values, refer to the BSON comparison order.

## Examples

### Basic Numeric Comparison

```javascript
// Find documents where quantity is greater than or equal to 100
db.inventory.find({
  quantity: { $gte: 100 }
})
```

### Date Comparison

```javascript
// Find documents with dates from January 1, 2024 onwards
db.events.find({
  date: {
    $gte: ISODate("2024-01-01")
  }
})
```

### Nested Field Comparison

```javascript
// Find documents where nested value is at least minimum threshold
db.metrics.find({
  "stats.value": { $gte: minThreshold }
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

1. **Inventory Management**
   ```javascript
   // Find products with sufficient stock
   db.products.find({
     stockLevel: { $gte: reorderPoint }
   })
   ```

2. **Date Range Queries**
   ```javascript
   // Find records from start date
   db.transactions.find({
     timestamp: {
       $gte: startDate
     }
   })
   ```

3. **Score Filtering**
   ```javascript
   // Find passing grades
   db.grades.find({
     score: { $gte: passingScore }
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

### Minimum Requirements

```javascript
// Find qualified candidates
db.candidates.find({
  experience: { $gte: minimumYears },
  rating: { $gte: minimumRating }
})
```

### Version Compatibility

```javascript
// Find compatible versions
db.software.find({
  version: { $gte: minVersion }
})
```

## Related Operators

- [$gt](gt.md) - Matches values greater than
- [$lt](lt.md) - Matches values less than
- [$lte](lte.md) - Matches values less than or equal to 