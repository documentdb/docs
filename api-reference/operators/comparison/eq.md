# $eq

Matches values that are equal to a specified value.

## Syntax

```javascript
{ <field>: { $eq: <value> } }
```

Or using the implicit equality operator:

```javascript
{ <field>: <value> }
```

## Description

The `$eq` operator matches documents where the value of a field equals the specified value. This operator provides explicit equality matching and is equivalent to the implicit equality syntax using `{ field: value }`.

## Examples

### Basic Equality Match

```javascript
// Find documents where status equals "active"
db.collection.find({
  status: { $eq: "active" }
})

// Equivalent implicit syntax
db.collection.find({
  status: "active"
})
```

### Matching Nested Fields

```javascript
// Find documents with specific nested field value
db.collection.find({
  "address.city": { $eq: "New York" }
})

// Equivalent implicit syntax
db.collection.find({
  "address.city": "New York"
})
```

### Array Element Matching

```javascript
// Find documents where tags array contains "urgent"
db.collection.find({
  tags: { $eq: "urgent" }
})

// Match exact array
db.collection.find({
  categories: { $eq: ["electronics", "phones"] }
})
```

## Behavior

1. **Type Handling**
   - Type-sensitive comparison
   - No type coercion
   - Exact matching

2. **Array Handling**
   - Matches array elements
   - Exact array matching
   - Order-sensitive

3. **Special Values**
   - Null values
   - Missing fields
   - Empty arrays

## Use Cases

1. **Exact Matching**
   ```javascript
   // Find users by exact email
   db.users.find({
     email: { $eq: "user@example.com" }
   })
   ```

2. **Status Checking**
   ```javascript
   // Find active orders
   db.orders.find({
     status: { $eq: "active" }
   })
   ```

3. **Category Filtering**
   ```javascript
   // Find products by category
   db.products.find({
     category: { $eq: "electronics" }
   })
   ```

## Performance Considerations

1. **Index Usage**
   - Efficiently uses indexes
   - Supports covered queries
   - Good for point queries

2. **Query Optimization**
   - Highly selective
   - Fast execution
   - Index-friendly

3. **Memory Usage**
   - Minimal memory overhead
   - Efficient comparison
   - No regex overhead

## Best Practices

1. **Index Design**
   - Create appropriate indexes
   - Consider compound indexes
   - Use for selective queries

2. **Query Structure**
   - Use implicit syntax when possible
   - Combine with other operators
   - Consider query patterns

3. **Data Consistency**
   - Maintain consistent data types
   - Handle case sensitivity
   - Consider collation

## Common Patterns

### User Authentication

```javascript
// Find user by username
db.users.find({
  username: { $eq: "johndoe" }
})
```

### Product Search

```javascript
// Find product by SKU
db.products.find({
  sku: { $eq: "ABC123" }
})
```

### Configuration Lookup

```javascript
// Find specific configuration
db.config.find({
  "settings.name": { $eq: "timezone" }
})
```

## Related Operators

- [$ne](ne.md) - Matches values that are not equal
- [$in](in.md) - Matches any value in an array
- [$nin](nin.md) - Matches none of the values in an array 