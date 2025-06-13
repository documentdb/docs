# Comparison Query Operators

Comparison query operators provide ways to compare values in queries. These operators allow you to match documents based on value comparisons.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$eq](eq.md) | Matches values that are equal to a specified value |
| [$gt](gt.md) | Matches values that are greater than a specified value |
| [$gte](gte.md) | Matches values that are greater than or equal to a specified value |
| [$in](in.md) | Matches any of the values specified in an array |
| [$lt](lt.md) | Matches values that are less than a specified value |
| [$lte](lte.md) | Matches values that are less than or equal to a specified value |
| [$ne](ne.md) | Matches values that are not equal to a specified value |
| [$nin](nin.md) | Matches none of the values specified in an array |

## Common Use Cases

- Value matching and filtering
- Range queries
- Set membership testing
- Exclusion filtering
- Numeric comparisons
- Date range queries
- Version comparison

## Example Usage

```javascript
// Equal to comparison
db.collection.find({
  age: { $eq: 25 }
})

// Greater than comparison
db.collection.find({
  price: { $gt: 100 }
})

// Range query (between values)
db.collection.find({
  quantity: {
    $gte: 10,
    $lte: 50
  }
})

// In array of values
db.collection.find({
  status: {
    $in: ["active", "pending"]
  }
})

// Not equal to comparison
db.collection.find({
  category: { $ne: "electronics" }
})

// Not in array of values
db.collection.find({
  type: {
    $nin: ["expired", "inactive"]
  }
})
```

## Best Practices

1. **Query Optimization**
   - Use appropriate indexes
   - Consider query selectivity
   - Combine operators efficiently

2. **Data Types**
   - Ensure consistent data types
   - Handle type conversion
   - Consider collation

3. **Performance**
   - Use covered queries
   - Leverage compound indexes
   - Consider query patterns

## Operator Details

### $eq
- Exact value matching
- Type-sensitive comparison
- Index-friendly queries

### $gt and $gte
- Range queries
- Numeric comparisons
- Date filtering

### $lt and $lte
- Upper bound queries
- Price range filtering
- Date constraints

### $in
- Multiple value matching
- Efficient OR conditions
- Array element matching

### $ne
- Exclusion filtering
- Non-matching queries
- Negative conditions

### $nin
- Multiple value exclusion
- Set complement operations
- Inverse membership tests

## Common Patterns

### Range Queries

```javascript
// Price range filter
db.products.find({
  price: {
    $gte: minPrice,
    $lte: maxPrice
  }
})
```

### Status Filtering

```javascript
// Multiple status match
db.orders.find({
  status: {
    $in: [
      "processing",
      "shipped",
      "delivered"
    ]
  }
})
```

### Exclusion Queries

```javascript
// Exclude specific categories
db.items.find({
  category: {
    $nin: [
      "archived",
      "deleted",
      "hidden"
    ]
  }
}) 