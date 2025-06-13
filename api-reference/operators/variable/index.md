# Variable Expression Operators

Variable expression operators allow you to define and use variables within aggregation expressions.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$let](let.md) | Defines variables for use within an expression |

## Common Use Cases

- Complex calculations
- Reusing values in expressions
- Improving expression readability
- Temporary variable storage
- Subexpression optimization

## Example Usage

```javascript
// Using variables in calculations
db.collection.aggregate([
  {
    $project: {
      total: {
        $let: {
          vars: {
            basePrice: "$price",
            taxRate: 0.08,
            shippingFee: 5.99
          },
          in: {
            $add: [
              "$$basePrice",
              { $multiply: ["$$basePrice", "$$taxRate"] },
              "$$shippingFee"
            ]
          }
        }
      }
    }
  }
])

// Complex document transformation
db.collection.aggregate([
  {
    $project: {
      result: {
        $let: {
          vars: {
            items: "$items",
            discount: { $cond: { if: "$isPremium", then: 0.1, else: 0 } }
          },
          in: {
            $map: {
              input: "$$items",
              as: "item",
              in: {
                name: "$$item.name",
                finalPrice: {
                  $subtract: [
                    "$$item.price",
                    { $multiply: ["$$item.price", "$$discount"] }
                  ]
                }
              }
            }
          }
        }
      }
    }
  }
])
```

## Best Practices

1. **Variable Naming**
   - Use descriptive names
   - Follow consistent conventions
   - Avoid reserved words

2. **Scope Management**
   - Understand variable scope
   - Clean up after usage
   - Avoid name conflicts

3. **Performance**
   - Minimize variable count
   - Reuse when appropriate
   - Consider memory usage

4. **Readability**
   - Document complex expressions
   - Break down large expressions
   - Use meaningful names 