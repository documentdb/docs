# $exp

Raises e to the specified exponent and returns the result.

## Syntax

```javascript
{ $exp: <exponent> }
```

## Description

The `$exp` operator calculates e (Euler's number, approximately 2.71828) raised to the specified exponent. This operator is commonly used in scientific, financial, and statistical calculations.

## Examples

### Basic Usage

```javascript
db.calculations.aggregate([
  {
    $project: {
      value: 1,
      exponential: { $exp: "$value" }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "value": 0 }    // e^0 = 1
{ "_id": 2, "value": 1 }    // e^1 ≈ 2.71828
{ "_id": 3, "value": 2 }    // e^2 ≈ 7.38906
{ "_id": 4, "value": -1 }   // e^-1 ≈ 0.36788
```

Output documents:
```javascript
{ "_id": 1, "value": 0, "exponential": 1 }
{ "_id": 2, "value": 1, "exponential": 2.718281828459045 }
{ "_id": 3, "value": 2, "exponential": 7.38905609893065 }
{ "_id": 4, "value": -1, "exponential": 0.36787944117144233 }
```

### Financial Calculations

```javascript
db.investments.aggregate([
  {
    $project: {
      principal: 1,
      rate: 1,
      time: 1,
      // Calculate continuous compound interest
      continuousInterest: {
        $multiply: [
          "$principal",
          { $exp: { $multiply: ["$rate", "$time"] } }
        ]
      }
    }
  }
])
```

### Growth Models

```javascript
db.population.aggregate([
  {
    $project: {
      initialPopulation: 1,
      growthRate: 1,
      time: 1,
      // Calculate population with exponential growth
      projectedPopulation: {
        $multiply: [
          "$initialPopulation",
          { $exp: { $multiply: ["$growthRate", "$time"] } }
        ]
      },
      // Calculate doubling time
      doublingTime: {
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
      temperature: 1,
      // Arrhenius equation for reaction rate
      reactionRate: {
        $multiply: [
          "$preExponentialFactor",
          {
            $exp: {
              $multiply: [
                -1,
                { $divide: ["$activationEnergy", { $multiply: 8.314, "$temperature" }] }
              ]
            }
          }
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Standard exponential calculation
   - Null: Returns null
   - Missing: Returns null
   - Non-numeric: Throws an error

2. **Precision**
   - Follows IEEE 754 standard
   - Maintains decimal precision
   - Returns exact values for common cases

3. **Special Cases**
   - Infinity: Returns Infinity
   - -Infinity: Returns 0
   - NaN: Returns NaN

## Use Cases

1. **Financial Mathematics**
   - Continuous compound interest
   - Present value calculations
   - Option pricing models
   - Growth projections

2. **Scientific Computing**
   - Chemical reaction rates
   - Physical decay processes
   - Natural growth models
   - Signal processing

3. **Statistical Analysis**
   - Probability distributions
   - Maximum likelihood estimation
   - Logistic regression
   - Risk assessment

4. **Data Science**
   - Feature scaling
   - Normalization
   - Machine learning models
   - Neural network activations

## Examples in Aggregation Pipeline

### Probability Calculations

```javascript
db.statistics.aggregate([
  {
    $project: {
      x: 1,
      // Normal distribution probability density function
      normalPDF: {
        $multiply: [
          { $divide: [1, { $sqrt: { $multiply: [2, 3.14159] } }] },
          {
            $exp: {
              $multiply: [
                -0.5,
                { $multiply: ["$x", "$x"] }
              ]
            }
          }
        ]
      }
    }
  }
])
```

### Risk Analysis

```javascript
db.portfolio.aggregate([
  {
    $project: {
      asset: 1,
      volatility: 1,
      // Value at Risk calculation using exponential
      valueAtRisk: {
        $multiply: [
          "$currentValue",
          {
            $subtract: [
              1,
              {
                $exp: {
                  $multiply: [
                    { $multiply: ["$volatility", 1.645] },  // 95% confidence
                    { $sqrt: { $divide: [1, 252] } }  // Daily VaR
                  ]
                }
              }
            ]
          }
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

3. **Computation**
   - CPU-intensive for large datasets
   - Consider batching calculations
   - Monitor for overflow conditions

## Best Practices

1. **Numeric Stability**
   - Handle large exponents carefully
   - Check for overflow conditions
   - Consider scaling inputs

2. **Error Handling**
   - Validate input ranges
   - Handle null values explicitly
   - Consider domain constraints

3. **Precision Management**
   - Define precision requirements
   - Use appropriate rounding
   - Document precision expectations

## Related Operators

- [$ln](ln.md) - Natural logarithm
- [$log](log.md) - Logarithm with base
- [$pow](pow.md) - Raises number to power
- [$sqrt](sqrt.md) - Square root 