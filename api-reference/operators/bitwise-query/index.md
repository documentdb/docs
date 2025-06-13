# Bitwise Query Operators

Bitwise query operators perform bit-level operations for querying documents. These operators allow you to find documents based on specific bit patterns in numeric fields.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$bitsAllClear](bitsallclear.md) | Matches numeric values where all specified bit positions are clear (0) |
| [$bitsAllSet](bitsallset.md) | Matches numeric values where all specified bit positions are set (1) |
| [$bitsAnyClear](bitsanyclear.md) | Matches numeric values where any of the specified bit positions are clear (0) |
| [$bitsAnySet](bitsanyset.md) | Matches numeric values where any of the specified bit positions are set (1) |

## Common Use Cases

- Permission and role checking
- Feature flag validation
- State verification
- Hardware status monitoring
- Configuration management
- Binary data filtering

## Example Usage

```javascript
// Check if all specified bits are clear (0)
db.collection.find({
  permissions: {
    $bitsAllClear: [1, 3]  // Check if bits 1 and 3 are both 0
  }
})

// Check if all specified bits are set (1)
db.collection.find({
  flags: {
    $bitsAllSet: [2, 4]  // Check if bits 2 and 4 are both 1
  }
})

// Check if any specified bits are clear (0)
db.collection.find({
  status: {
    $bitsAnyClear: [1, 2, 3]  // Check if any of bits 1, 2, or 3 is 0
  }
})

// Check if any specified bits are set (1)
db.collection.find({
  features: {
    $bitsAnySet: [0, 4, 7]  // Check if any of bits 0, 4, or 7 is 1
  }
})
```

## Best Practices

1. **Input Format**
   - Use array of bit positions
   - Use binary number (BinData)
   - Consider endianness

2. **Performance**
   - Index usage optimization
   - Query selectivity
   - Bit position ordering

3. **Data Consistency**
   - Validate bit positions
   - Handle null values
   - Document bit meanings

## Operator Details

### $bitsAllClear
- Checks if all specified bits are 0
- Useful for checking disabled features
- Common in permission validation

### $bitsAllSet
- Checks if all specified bits are 1
- Used for required feature verification
- Essential for state validation

### $bitsAnyClear
- Checks if any specified bit is 0
- Helpful for status monitoring
- Used in configuration checks

### $bitsAnySet
- Checks if any specified bit is 1
- Important for feature detection
- Used in capability checking 