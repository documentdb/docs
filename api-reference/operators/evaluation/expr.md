# $expr

The `$expr` operator allows the use of aggregation expressions within query language.

## Syntax

```javascript
{ $expr: <expression> }
```

## Description

The `$expr` operator:
- Enables the use of aggregation expressions in query conditions
- Allows comparisons between fields in the same document
- Supports the use of variables and conditional statements

## Examples

### Compare Two Fields

```javascript
// Find documents where quantity is greater than target
db.inventory.find({
   $expr: { $gt: ["$quantity", "$target"] }
})
```

### Using $cond with $expr

```javascript
// Find documents with conditional logic
db.orders.find({
   $expr: {
      $cond: {
         if: { $gt: ["$qty", 250] },
         then: { $lt: ["$price", 10] },
         else: { $lt: ["$price", 20] }
      }
   }
})
```

## Behavior

- `$expr` evaluates expressions during query execution
- Can reference document fields using the `$` prefix
- Supports most aggregation operators and expressions
- Cannot use regular query operators inside `$expr`

## Performance Considerations

- Using `$expr` may impact query performance as it evaluates expressions at runtime
- Indexes cannot be used effectively with `$expr` comparisons
- Consider using regular query operators when possible for better performance

## Related Operators

- [$cond](../conditional/cond.md) - Conditional operator used within expressions
- [$gt](../comparison/gt.md) - Greater than comparison operator 