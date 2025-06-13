# $ln

Calculates the natural logarithm (ln) of a number and returns the result.

## Syntax

```javascript
{ $ln: <number> }
```

## Description

The `$ln` operator calculates the natural logarithm (base e) of a number. The natural logarithm is the inverse of the exponential function (e^x). This operator is commonly used in scientific, financial, and statistical calculations.

## Examples

### Basic Usage

```javascript
db.calculations.aggregate([
  {
    $project: {
      value: 1,
      naturalLog: { $ln: "$value" }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "value": 1 }     // ln(1) = 0
{ "_id": 2, "value": 2.718281828459045 }  // ln(e) = 1
{ "_id": 3, "value": 10 }    // ln(10) ≈ 2.30259
{ "_id": 4, "value": 100 }   // ln(100) ≈ 4.60517
```

Output documents:
```javascript
{ "_id": 1, "value": 1, "naturalLog": 0 }
{ "_id": 2, "value": 2.718281828459045, "naturalLog": 1 }
{ "_id": 3, "value": 10, "naturalLog": 2.302585092994046 }
{ "_id": 4, "value": 100, "naturalLog": 4.605170185988092 }
```

### Financial Calculations

```javascript
db.investments.aggregate([
  {
    $project: {
      investment: 1,
      initialValue: 1,
      finalValue: 1,
      time: 1,
      // Calculate continuous growth rate
      continuousRate: {
        $divide: [
          { $ln: { $divide: ["$finalValue", "$initialValue"] } },
          "$time"
        ]
      },
      // Calculate time to double
      timeToDouble: {
        $divide: [
          { $ln: 2 },
          "$growthRate"
        ]
      }
    }
  }
])
```

### Scientific Calculations

```javascript
db.reactions.aggregate([
  {
    $project: {
      reaction: 1,
      concentration: 1,
      // Calculate pH from hydrogen ion concentration
      pH: {
        $multiply: [
          -1,
          { $ln: { $divide: ["$concentration", 1e-7] } }
        ]
      },
      // Calculate reaction order
      reactionOrder: {
        $divide: [
          { $ln: { $divide: ["$rate2", "$rate1"] } },
          { $ln: { $divide: ["$concentration2", "$concentration1"] } }
        ]
      }
    }
  }
])
```

### Statistical Analysis

```javascript
db.statistics.aggregate([
  {
    $project: {
      dataset: 1,
      // Calculate log-likelihood
      logLikelihood: {
        $sum: {
          $map: {
            input: "$observations",
            as: "x",
            in: {
              $ln: {
                $multiply: [
                  { $sqrt: { $multiply: [2, 3.14159] } },
                  { $exp: { 
                      $multiply: [
                        -0.5,
                        { $pow: [{ $subtract: ["$$x", "$mean"] }, 2] }
                      ]
                    }
                  }
                ]
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

1. **Input Types**
   - Positive numbers: Returns natural logarithm
   - Zero or negative: Returns null
   - Null: Returns null
   - Missing: Returns null
   - Non-numeric: Throws an error

2. **Precision**
   - Follows IEEE 754 standard
   - Maintains decimal precision
   - Returns exact values for common cases

3. **Special Cases**
   - Input ≤ 0: Returns null
   - Infinity: Returns Infinity
   - NaN: Returns NaN

## Use Cases

1. **Financial Analysis**
   - Growth rate calculations
   - Compound interest analysis
   - Investment returns
   - Time value calculations

2. **Scientific Computing**
   - pH calculations
   - Reaction kinetics
   - Decay analysis
   - Energy calculations

3. **Statistical Analysis**
   - Log-likelihood computations
   - Information theory
   - Entropy calculations
   - Distribution analysis

4. **Data Science**
   - Feature transformation
   - Dimensionality reduction
   - Model optimization
   - Scale normalization

## Examples in Aggregation Pipeline

### Growth Analysis

```javascript
db.growth.aggregate([
  {
    $project: {
      period: 1,
      // Calculate compound annual growth rate (CAGR)
      cagr: {
        $multiply: [
          {
            $subtract: [
              { $exp: {
                  $divide: [
                    { $ln: { $divide: ["$endValue", "$startValue"] } },
                    "$years"
                  ]
                }
              },
              1
            ]
          },
          100
        ]
      }
    }
  }
])
```

### Information Theory

```javascript
db.information.aggregate([
  {
    $project: {
      event: 1,
      probability: 1,
      // Calculate information content (in bits)
      informationContent: {
        $divide: [
          { $multiply: [-1, { $ln: "$probability" }] },
          { $ln: 2 }
        ]
      },
      // Calculate entropy
      entropy: {
        $multiply: [
          "$probability",
          { $ln: { $divide: [1, "$probability"] } }
        ]
      }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Combine with other operations efficiently
   - Consider pre-computing common values
   - Use appropriate numeric precision

2. **Memory Usage**
   - Minimal memory overhead
   - Efficient for large datasets
   - Consider result size limitations

3. **Domain Validation**
   - Check for positive inputs
   - Handle edge cases appropriately
   - Consider numerical stability

## Best Practices

1. **Input Validation**
   - Ensure positive input values
   - Handle zero and negative cases
   - Consider domain constraints

2. **Error Handling**
   - Handle undefined results
   - Provide meaningful error messages
   - Consider fallback values

3. **Precision Management**
   - Define precision requirements
   - Use appropriate rounding
   - Document precision expectations

## Related Operators

- [$exp](exp.md) - Exponential function
- [$log](log.md) - Logarithm with base
- [$log10](log10.md) - Base-10 logarithm
- [$pow](pow.md) - Power function 