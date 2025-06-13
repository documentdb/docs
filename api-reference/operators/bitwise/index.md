# Bitwise Operators

Bitwise operators perform operations on integer values at the bit level. These operators allow you to manipulate individual bits within numeric fields.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$bitand](bitand.md) | Performs a bitwise AND operation between two numbers |
| [$bitnot](bitnot.md) | Performs a bitwise NOT operation on a number |
| [$bitor](bitor.md) | Performs a bitwise OR operation between two numbers |
| [$bitxnor](bitxnor.md) | Performs a bitwise XNOR operation between two numbers |

## Common Use Cases

- Flag and permission management
- Bit field manipulation
- Binary data processing
- Hardware interface operations

## Example Usage

```javascript
// Using $bitand to check if a flag is set
db.collection.find({
  permissions: {
    $bitand: [4, 4]  // Check if bit 2 is set (1 << 2 = 4)
  }
})

// Using $bitor to set flags
db.collection.update(
  { _id: 1 },
  {
    $set: {
      flags: {
        $bitor: [currentFlags, newFlag]
      }
    }
  }
)

// Using $bitnot to invert bits
db.collection.find({
  value: {
    $bitnot: 5  // Inverts all bits in 5
  }
})
```

## Best Practices

1. **Type Handling**
   - Use with integer values only
   - Consider sign bit implications
   - Handle overflow cases

2. **Performance**
   - Efficient for bit manipulation
   - Use appropriate indexes
   - Consider query optimization

3. **Data Consistency**
   - Validate input values
   - Handle edge cases
   - Document bit field meanings

## Operator Details

### $bitand
- Performs AND operation on each bit
- Returns 1 only when both bits are 1
- Common for permission checking

### $bitnot
- Inverts all bits in the number
- Changes 0s to 1s and 1s to 0s
- Useful for bit masking

### $bitor
- Performs OR operation on each bit
- Returns 1 if either bit is 1
- Used for combining flags

### $bitxnor
- Performs XNOR operation on each bit
- Returns 1 when bits are the same
- Used for bit pattern matching 