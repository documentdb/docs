# $let

Defines variables for use within an expression.

## Syntax

```javascript
{
  $let: {
    vars: { <var1>: <expression1>, ... },
    in: <expression>
  }
}
```

## Description

The `$let` operator binds variables for use in a specified expression and returns the result. Variables can be used within the `in` expression by prefixing them with `$$`.

## Examples

### Basic Variable Usage

```javascript
db.orders.aggregate([
  {
    $project: {
      finalPrice: {
        $let: {
          vars: {
            basePrice: "$price",
            taxRate: 0.08,
            discount: { $cond: { if: "$isPremium", then: 0.1, else: 0 } }
          },
          in: {
            $subtract: [
              { $multiply: ["$$basePrice", { $add: [1, "$$taxRate"] }] },
              { $multiply: ["$$basePrice", "$$discount"] }
            ]
          }
        }
      }
    }
  }
])
```

### Complex Calculations

```javascript
db.inventory.aggregate([
  {
    $project: {
      item: 1,
      profitAnalysis: {
        $let: {
          vars: {
            cost: "$manufacturingCost",
            price: "$sellingPrice",
            overhead: "$overheadCost",
            volume: "$monthlyVolume"
          },
          in: {
            totalRevenue: { $multiply: ["$$price", "$$volume"] },
            totalCost: { $multiply: [{ $add: ["$$cost", "$$overhead"] }, "$$volume"] },
            profitMargin: {
              $divide: [
                { $subtract: ["$$price", { $add: ["$$cost", "$$overhead"] }] },
                "$$price"
              ]
            }
          }
        }
      }
    }
  }
])
```

### Nested Variables

```javascript
db.sales.aggregate([
  {
    $project: {
      analysis: {
        $let: {
          vars: {
            baseMetrics: {
              revenue: "$amount",
              cost: "$cost",
              tax: { $multiply: ["$amount", 0.08] }
            }
          },
          in: {
            $let: {
              vars: {
                grossProfit: { $subtract: ["$$baseMetrics.revenue", "$$baseMetrics.cost"] }
              },
              in: {
                grossProfit: "$$grossProfit",
                netProfit: { $subtract: ["$$grossProfit", "$$baseMetrics.tax"] },
                margin: {
                  $divide: ["$$grossProfit", "$$baseMetrics.revenue"]
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

## Behavior

1. **Variable Scope**
   - Limited to `in` expression
   - Can reference other variables
   - Supports nested scopes
   - Follows lexical scoping

2. **Expression Evaluation**
   - Variables evaluated first
   - Results used in expression
   - Maintains type information
   - Supports all expression operators

3. **Error Handling**
   - Undefined variables
   - Type mismatches
   - Invalid expressions
   - Scope violations

## Use Cases

1. **Complex Calculations**
   - Financial computations
   - Statistical analysis
   - Multi-step processing
   - Formula evaluation

2. **Code Organization**
   - Breaking down logic
   - Reusing values
   - Improving readability
   - Reducing duplication

3. **Data Transformation**
   - Document restructuring
   - Field computation
   - Conditional processing
   - Value normalization

## Best Practices

1. **Variable Naming**
   - Use descriptive names
   - Follow conventions
   - Avoid reserved words
   - Document purpose

2. **Expression Structure**
   - Break down complex logic
   - Organize calculations
   - Consider readability
   - Plan variable usage

3. **Performance**
   - Minimize variable count
   - Optimize expressions
   - Consider memory usage
   - Monitor execution time

## Related Operators

None - The `$let` operator is a standalone expression operator. 