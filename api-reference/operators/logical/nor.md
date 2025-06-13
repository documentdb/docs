# $nor

The `$nor` operator performs a logical NOR operation on an array of expressions and selects documents that fail all the expressions.

## Syntax

```javascript
{ $nor: [ <expression1>, <expression2>, ... ] }
```

## Description

The `$nor` operator:
- Takes an array of expressions
- Returns documents that fail to match all conditions
- Essentially performs a logical NOT operation on a logical OR of the conditions
- Returns all documents that do not match any of the conditions

## Examples

### Basic NOR Operation

```javascript
// Find documents where price is not greater than 100 AND quantity is not less than 20
db.inventory.find({
   $nor: [
      { price: { $gt: 100 } },
      { qty: { $lt: 20 } }
   ]
})
```

### Checking for Field Existence

```javascript
// Find documents where neither field exists
db.products.find({
   $nor: [
      { price: { $exists: true } },
      { qty: { $exists: true } }
   ]
})
```

### Complex Conditions

```javascript
// Find documents that don't match any of several conditions
db.orders.find({
   $nor: [
      { status: "cancelled" },
      { total: { $lt: 50 } },
      { "items.qty": { $gt: 100 } }
   ]
})
```

## Behavior

- Returns documents that fail all specified conditions
- Can be used to implement complex negative conditions
- Useful for finding documents that don't match any of several criteria
- Each condition in the array is evaluated independently

## Performance Considerations

- May require full collection scan if conditions can't use indexes effectively
- Consider using compound indexes when multiple fields are involved
- More complex to optimize than simple AND/OR operations
- Use sparingly as it can make queries harder to understand and maintain

## Related Operators

- [$and](and.md) - Logical AND operator
- [$or](or.md) - Logical OR operator
- [$not](not.md) - Logical NOT operator 