# $log

Calculates the logarithm of a number in the specified base.

## Syntax

```javascript
{
  $log: [
    <number>,
    <base>
  ]
}
```

## Description

The `$log` operator calculates the logarithm of a number using the specified base. It takes two parameters: the number and the base. This operator is useful for various mathematical, scientific, and data analysis calculations.

## Examples

### Basic Usage

```javascript
db.calculations.aggregate([
  {
    $project: {
      value: 1,
      base: 1,
      logarithm: { $log: ["$value", "$base"] }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "value": 100, "base": 10 }   // log₁₀(100) = 2
{ "_id": 2, "value": 8, "base": 2 }      // log₂(8) = 3
{ "_id": 3, "value": 16, "base": 4 }     // log₄(16) = 2
{ "_id": 4, "value": 1, "base": 5 }      // log₅(1) = 0
```

Output documents:
```javascript
{ "_id": 1, "value": 100, "base": 10, "logarithm": 2 }
{ "_id": 2, "value": 8, "base": 2, "logarithm": 3 }
{ "_id": 3, "value": 16, "base": 4, "logarithm": 2 }
{ "_id": 4, "value": 1, "base": 5, "logarithm": 0 }
```

### Change of Base Calculations

```javascript
db.conversions.aggregate([
  {
    $project: {
      number: 1,
      // Convert between different logarithm bases
      log2: { $log: ["$number", 2] },
      log10: { $log: ["$number", 10] },
      // Change of base formula demonstration
      log5: {
        $divide: [
          { $ln: "$number" },
          { $ln: 5 }
        ]
      }
    }
  }
])
```

### Scientific Calculations

```javascript
db.measurements.aggregate([
  {
    $project: {
      reading: 1,
      // Calculate decibels (dB)
      decibels: {
        $multiply: [
          20,
          { $log: [{ $abs: { $divide: ["$amplitude", "$referenceAmplitude"] } }, 10] }
        ]
      },
      // Calculate pH scale
      pH: {
        $multiply: [
          -1,
          { $log: ["$hydrogenConcentration", 10] }
        ]
      }
    }
  }
])
```

### Information Theory

```javascript
db.signals.aggregate([
  {
    $project: {
      signal: 1,
      // Calculate information content in bits
      bitsOfInformation: {
        $log: [{ $divide: [1, "$probability"] }, 2]
      },
      // Calculate channel capacity
      channelCapacity: {
        $multiply: [
          "$bandwidth",
          { $log: [{ $add: [1, { $divide: ["$signalPower", "$noisePower"] }] }, 2] }
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Returns logarithm
   - Base must be positive and not 1
   - Number must be positive
   - Null: Returns null
   - Missing: Returns null

2. **Precision**
   - Follows IEEE 754 standard
   - Maintains decimal precision
   - Returns exact values for perfect powers

3. **Special Cases**
   - Input ≤ 0: Returns null
   - Base ≤ 0 or = 1: Returns null
   - Infinity: Returns Infinity
   - NaN: Returns NaN

## Use Cases

1. **Scientific Computing**
   - Decibel calculations
   - pH measurements
   - Sound intensity
   - Energy levels

2. **Information Theory**
   - Entropy calculation
   - Information content
   - Channel capacity
   - Data compression

3. **Data Analysis**
   - Scale transformations
   - Data normalization
   - Pattern recognition
   - Signal processing

4. **Financial Analysis**
   - Growth rate analysis
   - Risk assessment
   - Market indicators
   - Trend analysis

## Examples in Aggregation Pipeline

### Scale Transformation

```javascript
db.analysis.aggregate([
  {
    $project: {
      value: 1,
      // Log transform with different bases
      transforms: {
        binary: { $log: ["$value", 2] },
        natural: { $ln: "$value" },
        decimal: { $log: ["$value", 10] },
        custom: { $log: ["$value", "$customBase"] }
      },
      // Normalize log values
      normalized: {
        $divide: [
          { $log: ["$value", 10] },
          { $log: ["$maxValue", 10] }
        ]
      }
    }
  }
])
```

### Signal Processing

```javascript
db.signals.aggregate([
  {
    $project: {
      frequency: 1,
      amplitude: 1,
      // Calculate power in decibels
      powerDB: {
        $multiply: [
          10,
          { $log: [
              { $divide: ["$power", "$referencePower"] },
              10
            ]
          }
        ]
      },
      // Calculate frequency ratios
      frequencyRatio: {
        $log: [
          { $divide: ["$frequency", "$referenceFrequency"] },
          2
        ]
      }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Use appropriate base for calculations
   - Consider pre-computing common values
   - Combine operations efficiently

2. **Memory Usage**
   - Minimal memory overhead
   - Efficient for large datasets
   - Consider result size limitations

3. **Computation**
   - More expensive than basic arithmetic
   - Consider caching frequent calculations
   - Use appropriate precision

## Best Practices

1. **Input Validation**
   - Verify positive input values
   - Validate base selection
   - Handle edge cases

2. **Base Selection**
   - Choose appropriate base for domain
   - Document base selection rationale
   - Consider computational efficiency

3. **Error Handling**
   - Handle invalid inputs gracefully
   - Provide meaningful error messages
   - Consider fallback values

## Related Operators

- [$ln](ln.md) - Natural logarithm
- [$log10](log10.md) - Base-10 logarithm
- [$exp](exp.md) - Exponential function
- [$pow](pow.md) - Power function 