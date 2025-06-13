# $bitxnor

Performs a bitwise XNOR (exclusive NOR) operation between two numbers.

## Syntax

```javascript
{ <field>: { $bitxnor: [ <number1>, <number2> ] } }
```

## Description

The `$bitxnor` operator performs a bitwise XNOR operation between two integers. It compares each bit of the first operand to the corresponding bit of the second operand. The result bit is set to 1 if both bits are the same (both 0 or both 1); otherwise, it is set to 0.

## Examples

### Basic Bitwise XNOR

```javascript
// Initial document
{ _id: 1, pattern: 5 }  // 5 in binary is 0101

// Find documents where pattern XNOR with 3 (0011) matches
db.collection.find({
  pattern: {
    $bitxnor: [5, 3]  // Result: -7 (1001 in two's complement)
  }
})
```

### Pattern Matching

```javascript
// Initial document
{
  _id: 2,
  template: 15  // 1111 in binary
}

// Find matching patterns
db.collection.find({
  template: {
    $bitxnor: ["$template", 15]  // Match exact patterns
  }
})
```

### Similarity Check

```javascript
// Initial documents
{
  _id: 3,
  signal1: 170,  // 10101010 in binary
  signal2: 165   // 10100101 in binary
}

// Check bit pattern similarity
db.collection.find({
  $expr: {
    $gt: [
      { $bitxnor: ["$signal1", "$signal2"] },
      0  // Positive result indicates more matching bits
    ]
  }
})
```

## Behavior

1. **Input Requirements**
   - Accepts integer values
   - Converts to 32-bit integers
   - Handles negative numbers

2. **Result Type**
   - Returns 32-bit integer
   - Uses two's complement
   - Preserves sign bit

3. **Special Cases**
   - Null or missing fields
   - Non-integer values
   - Array operands

## Use Cases

1. **Pattern Matching**
   - Similarity detection
   - Error checking
   - Signal comparison

2. **Data Validation**
   - Bit pattern verification
   - Data integrity checks
   - Format validation

3. **Signal Processing**
   - Digital signal analysis
   - Pattern recognition
   - Noise detection

## Performance Considerations

1. **Operation Cost**
   - Two CPU instructions
   - Constant time operation
   - No memory allocation

2. **Index Usage**
   - Can use indexes
   - Consider selectivity
   - Plan query optimization

3. **Query Planning**
   - Combine with other operators
   - Use in aggregation
   - Consider alternatives

## Best Practices

1. **Input Validation**
   - Check value ranges
   - Handle edge cases
   - Document bit meanings

2. **Error Handling**
   - Handle null values
   - Check field existence
   - Validate input types

3. **Documentation**
   - Document bit patterns
   - Explain comparisons
   - Maintain bit maps

## Common Patterns

### Pattern Verification

```javascript
// Verify bit pattern matches template
db.collection.find({
  data: {
    $bitxnor: [
      "$data",
      PATTERN_TEMPLATE
    ]
  }
})
```

### Similarity Detection

```javascript
// Find similar patterns within threshold
db.patterns.find({
  $expr: {
    $gte: [
      { $bitxnor: ["$pattern", targetPattern] },
      similarityThreshold
    ]
  }
})
```

## Related Operators

- [$bitand](bitand.md) - Performs bitwise AND
- [$bitor](bitor.md) - Performs bitwise OR
- [$bitnot](bitnot.md) - Performs bitwise NOT 