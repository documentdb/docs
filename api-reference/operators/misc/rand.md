# $rand

Generates a random float between 0 and 1.

## Syntax

```javascript
{ $rand: {} }
```

## Description

The `$rand` operator generates a random floating-point number between 0 and 1 (inclusive of 0, but not 1). This operator is useful for random sampling, probabilistic operations, and testing scenarios.

## Examples

### Basic Random Value

```javascript
db.collection.aggregate([
  {
    $project: {
      randomValue: { $rand: {} }
    }
  }
])
```

### Random Sampling

```javascript
// Select approximately 10% of documents
db.collection.aggregate([
  {
    $match: {
      $expr: {
        $lt: [{ $rand: {} }, 0.1]  // 10% probability
      }
    }
  }
])
```

### Random Assignment

```javascript
// Randomly assign documents to test or control group
db.users.aggregate([
  {
    $set: {
      experimentGroup: {
        $cond: {
          if: { $lt: [{ $rand: {} }, 0.5] },
          then: "test",
          else: "control"
        }
      }
    }
  }
])
```

### Weighted Random Selection

```javascript
db.products.aggregate([
  {
    $project: {
      name: 1,
      isSelected: {
        $cond: {
          if: { $lt: [{ $rand: {} }, "$selectionWeight"] },
          then: true,
          else: false
        }
      }
    }
  }
])
```

## Behavior

1. **Value Range**
   - Returns float between 0 and 1
   - Includes 0
   - Excludes 1
   - Uniform distribution

2. **Execution Context**
   - New random value per document
   - Independent between calls
   - Non-deterministic results

3. **Performance Impact**
   - Minimal overhead
   - No caching of values
   - Scales with document count

## Use Cases

1. **Data Sampling**
   - Random document selection
   - Statistical sampling
   - A/B testing
   - Load testing

2. **Probabilistic Operations**
   - Random assignments
   - Weighted selections
   - Probability-based filtering
   - Randomized testing

3. **Simulation**
   - Data generation
   - Monte Carlo methods
   - Random behavior testing
   - Performance testing

## Best Practices

1. **Sampling**
   - Consider data distribution
   - Validate sample sizes
   - Account for edge cases
   - Document sampling rates

2. **Performance**
   - Use indexes when possible
   - Consider result set size
   - Optimize pipeline stages
   - Monitor resource usage

3. **Reproducibility**
   - Document random seeds
   - Log selection criteria
   - Track sampling parameters
   - Consider deterministic alternatives

## Related Operators

- [$sample](../aggregation/sample.md) - Randomly selects specified number of documents 