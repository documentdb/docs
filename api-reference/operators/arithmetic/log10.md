# $log10

Calculates the log base 10 of a number and returns the result.

## Syntax

```javascript
{ $log10: <number> }
```

## Description

The `$log10` operator calculates the logarithm of a number in base 10. This is equivalent to using `$log` with a base of 10 but is more convenient and potentially more efficient for base-10 calculations.

## Examples

### Basic Usage

```javascript
db.calculations.aggregate([
  {
    $project: {
      value: 1,
      log10: { $log10: "$value" }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "value": 1 }      // log₁₀(1) = 0
{ "_id": 2, "value": 10 }     // log₁₀(10) = 1
{ "_id": 3, "value": 100 }    // log₁₀(100) = 2
{ "_id": 4, "value": 1000 }   // log₁₀(1000) = 3
```

Output documents:
```javascript
{ "_id": 1, "value": 1, "log10": 0 }
{ "_id": 2, "value": 10, "log10": 1 }
{ "_id": 3, "value": 100, "log10": 2 }
{ "_id": 4, "value": 1000, "log10": 3 }
```

### Scientific Calculations

```javascript
db.measurements.aggregate([
  {
    $project: {
      reading: 1,
      // Calculate pH value
      pH: {
        $multiply: [
          -1,
          { $log10: "$hydrogenConcentration" }
        ]
      },
      // Calculate decibels
      decibels: {
        $multiply: [
          20,
          { $log10: { $divide: ["$amplitude", "$referenceAmplitude"] } }
        ]
      }
    }
  }
])
```

### Scale Transformations

```javascript
db.data.aggregate([
  {
    $project: {
      value: 1,
      // Log scale transformation
      logScale: { $log10: "$value" },
      // Normalize to range [0,1] using log scale
      normalized: {
        $divide: [
          { $log10: "$value" },
          { $log10: "$maxValue" }
        ]
      },
      // Calculate order of magnitude
      magnitude: {
        $floor: { $log10: "$value" }
      }
    }
  }
])
```

### Range Analysis

```javascript
db.ranges.aggregate([
  {
    $project: {
      range: 1,
      // Calculate dynamic range in decibels
      dynamicRange: {
        $multiply: [
          20,
          {
            $log10: {
              $divide: ["$maxValue", "$minValue"]
            }
          }
        ]
      },
      // Calculate number of digits
      significantDigits: {
        $add: [
          1,
          {
            $floor: { $log10: "$value" }
          }
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Positive numbers: Returns log base 10
   - Zero or negative: Returns null
   - Null: Returns null
   - Missing: Returns null
   - Non-numeric: Throws an error

2. **Precision**
   - Follows IEEE 754 standard
   - Maintains decimal precision
   - Returns exact values for powers of 10

3. **Special Cases**
   - Input ≤ 0: Returns null
   - Infinity: Returns Infinity
   - NaN: Returns NaN

## Use Cases

1. **Scientific Measurements**
   - pH calculations
   - Decibel measurements
   - Richter scale
   - Light intensity

2. **Data Analysis**
   - Scale transformation
   - Order of magnitude
   - Range compression
   - Dynamic range

3. **Engineering**
   - Signal processing
   - Power calculations
   - Frequency analysis
   - Gain measurements

4. **Visualization**
   - Log scale plots
   - Data normalization
   - Range compression
   - Pattern analysis

## Examples in Aggregation Pipeline

### Signal Analysis

```javascript
db.signals.aggregate([
  {
    $project: {
      signal: 1,
      // Calculate power level in dB
      powerLevel: {
        $multiply: [
          10,
          { $log10: { $divide: ["$power", 1e-3] } }  // Reference to 1mW
        ]
      },
      // Calculate signal-to-noise ratio
      snr: {
        $multiply: [
          10,
          { $log10: { $divide: ["$signalPower", "$noisePower"] } }
        ]
      }
    }
  }
])
```

### Data Scaling

```javascript
db.values.aggregate([
  {
    $project: {
      value: 1,
      // Log scale binning
      bin: {
        $floor: { $log10: "$value" }
      },
      // Normalize to percentage of maximum
      percentage: {
        $multiply: [
          {
            $divide: [
              { $log10: "$value" },
              { $log10: "$maxValue" }
            ]
          },
          100
        ]
      }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - More efficient than `$log` with base 10
   - Good for large-scale calculations
   - Consider pre-computing for repeated use

2. **Memory Usage**
   - Minimal memory overhead
   - Efficient for large datasets
   - Consider result size limitations

3. **Computation**
   - Faster than general logarithm
   - Optimized for base 10
   - Good for repeated calculations

## Best Practices

1. **Input Validation**
   - Check for positive values
   - Handle zero and negative cases
   - Consider domain constraints

2. **Error Handling**
   - Handle undefined results
   - Provide meaningful errors
   - Consider fallback values

3. **Scale Selection**
   - Consider data range
   - Document scale choice
   - Handle extreme values

## Related Operators

- [$log](log.md) - Logarithm with custom base
- [$ln](ln.md) - Natural logarithm
- [$exp](exp.md) - Exponential function
- [$pow](pow.md) - Power function 