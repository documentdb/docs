# $not

The `$not` operator performs a logical NOT operation on a specified expression and selects documents that do not match the expression.

## Syntax

```javascript
{ field: { $not: { <operator-expression> } } }
```

## Description

The `$not` operator:
- Performs a logical NOT operation on a single expression
- Must be applied to a specific field
- Works with regular expressions and other operator expressions
- Cannot directly contain a list of conditions

## Examples

### Basic NOT Operation

```javascript
// Find documents where price is not greater than 100
db.inventory.find({
   price: { $not: { $gt: 100 } }
})
```

### With Regular Expressions

```javascript
// Find documents where name does not match pattern
db.products.find({
   name: { $not: /^test/ }
})
```

### Complex Conditions

```javascript
// Find documents where status field doesn't match specific criteria
db.orders.find({
   status: { 
      $not: { 
         $in: ["cancelled", "pending"] 
      }
   }
})
```

## Behavior

- Inverts the effect of an operator expression
- Cannot be applied directly to a value (must use an operator expression)
- Useful for negating complex conditions on a single field
- Often used with regular expressions and comparison operators

## Performance Considerations

- Can use indexes when applied to indexed fields
- May require full collection scan for complex expressions
- Consider using `$ne` or `$nin` for simple inequality matches
- Regular expression performance depends on pattern complexity

## Common Use Cases

- Negating regular expression patterns
- Inverting comparison operations
- Excluding documents based on complex field criteria
- Pattern matching exclusions

## Related Operators

- [$and](and.md) - Logical AND operator
- [$or](or.md) - Logical OR operator
- [$nor](nor.md) - Logical NOR operator
- [$ne](../comparison/ne.md) - Not equal comparison operator 