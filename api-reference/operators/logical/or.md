# $or

The `$or` operator performs a logical OR operation on an array of expressions and selects documents that satisfy at least one of the expressions.

## Syntax

```javascript
{ $or: [ <expression1>, <expression2>, ... ] }
```

## Description

The `$or` operator:
- Takes an array of expressions
- Returns documents that match any of the conditions
- Can combine different types of expressions
- Useful for querying alternative conditions

## Examples

### Basic OR Operation

```javascript
// Find documents where price is less than 100 OR quantity is greater than 20
db.inventory.find({
   $or: [
      { price: { $lt: 100 } },
      { qty: { $gt: 20 } }
   ]
})
```

### Multiple Field Conditions

```javascript
// Find documents matching multiple field conditions
db.products.find({
   $or: [
      { category: "electronics" },
      { price: { $lt: 100 } },
      { "reviews.rating": { $gt: 4.5 } }
   ]
})
```

### Combining with Other Logical Operators

```javascript
// Find active orders that are either high value or high quantity
db.orders.find({
   status: "active",
   $or: [
      { total: { $gt: 1000 } },
      { "items.qty": { $gt: 50 } }
   ]
})
```

## Behavior

- Short-circuits evaluation when a matching condition is found
- Evaluates conditions from left to right
- Can use different operators in each condition
- Returns documents that match any of the conditions

## Performance Considerations

- Can use indexes effectively when conditions are properly indexed
- Order conditions from most to least selective for better performance
- Consider using compound indexes for frequently combined conditions
- May require multiple index scans for different conditions

## Common Use Cases

- Searching across multiple fields
- Implementing alternative matching criteria
- Combining different types of conditions
- Building flexible search queries

## Related Operators

- [$and](and.md) - Logical AND operator
- [$not](not.md) - Logical NOT operator
- [$nor](nor.md) - Logical NOR operator
- [$in](../comparison/in.md) - Match any value in an array 