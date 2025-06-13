# $bitand

Performs a bitwise AND operation between two numbers.

## Syntax

```javascript
{ <field>: { $bitand: [ <number1>, <number2> ] } }
```

## Description

The `$bitand` operator performs a bitwise AND operation between two integers. It compares each bit of the first operand to the corresponding bit of the second operand. If both bits are 1, the corresponding result bit is set to 1; otherwise, it is set to 0.

## Examples

### Basic Bitwise AND

```javascript
// Initial document
{ _id: 1, permissions: 7 }  // 7 in binary is 0111

// Find documents where permissions has bit pattern matching 6 (0110)
db.collection.find({
  permissions: {
    $bitand: [7, 6]  // Result: 6 (0110)
  }
})
```

### Permission Checking

```javascript
// Initial document
{
  _id: 2,
  flags: 13  // 1101 in binary
}

// Check if specific bits are set
db.collection.find({
  flags: {
    $bitand: [13, 4]  // Check if bit 2 is set (1 << 2 = 4)
  }
})
```

### Multiple Field Comparison

```javascript
// Initial documents
{
  _id: 3,
  value1: 5,   // 0101 in binary
  value2: 3    // 0011 in binary
}

// Find documents where bitwise AND of fields matches
db.collection.find({
  $expr: {
    $eq: [
      { $bitand: ["$value1", "$value2"] },
      1  // 0001 in binary
    ]
  }
})
```

## Behavior

1. **Input Requirements**
   - Accepts integer values
   - Converts numbers to 32-bit integers
   - Handles negative numbers

2. **Result Type**
   - Returns 32-bit integer
   - Preserves sign bit
   - Follows two's complement

3. **Special Cases**
   - Null or missing fields
   - Non-integer values
   - Array operands

## Use Cases

1. **Permission Management**
   - Role-based access control
   - Feature flags
   - Security masks

2. **Hardware Interface**
   - Register manipulation
   - Device control
   - Status checking

3. **Data Processing**
   - Bit field extraction
   - Pattern matching
   - Flag combination

## Performance Considerations

1. **Operation Cost**
   - Single CPU instruction
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
   - Explain flag meanings
   - Maintain bit maps

## Common Patterns

### Permission Check

```javascript
// Check if user has all required permissions
db.users.find({
  permissions: {
    $bitand: [
      "$permissions",
      requiredPermissions
    ]
  }
})
```

### Feature Flags

```javascript
// Check if specific features are enabled
db.config.find({
  features: {
    $bitand: [
      "$features",
      FEATURE_FLAG
    ]
  }
})
```

## Related Operators

- [$bitor](bitor.md) - Performs bitwise OR
- [$bitnot](bitnot.md) - Performs bitwise NOT
- [$bitxnor](bitxnor.md) - Performs bitwise XNOR 