# $bitor

Performs a bitwise OR operation between two numbers.

## Syntax

```javascript
{ <field>: { $bitor: [ <number1>, <number2> ] } }
```

## Description

The `$bitor` operator performs a bitwise OR operation between two integers. It compares each bit of the first operand to the corresponding bit of the second operand. If either bit is 1, the corresponding result bit is set to 1; otherwise, it is set to 0.

## Examples

### Basic Bitwise OR

```javascript
// Initial document
{ _id: 1, flags: 5 }  // 5 in binary is 0101

// Find documents where flags combined with 3 (0011) match
db.collection.find({
  flags: {
    $bitor: [5, 3]  // Result: 7 (0111)
  }
})
```

### Permission Combination

```javascript
// Initial document
{
  _id: 2,
  permissions: 12  // 1100 in binary
}

// Add new permission bit
db.collection.update(
  { _id: 2 },
  {
    $set: {
      permissions: {
        $bitor: ["$permissions", 1]  // Add bit 0 (0001)
      }
    }
  }
)
```

### Flag Merging

```javascript
// Initial documents
{
  _id: 3,
  flags1: 10,  // 1010 in binary
  flags2: 5    // 0101 in binary
}

// Combine flags
db.collection.find({
  $expr: {
    $eq: [
      { $bitor: ["$flags1", "$flags2"] },
      15  // 1111 in binary
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
   - Preserves sign bit
   - Follows two's complement

3. **Special Cases**
   - Null or missing fields
   - Non-integer values
   - Array operands

## Use Cases

1. **Permission Management**
   - Combining access rights
   - Setting feature flags
   - Merging capabilities

2. **Flag Operations**
   - Setting bit flags
   - Combining states
   - Feature activation

3. **Data Combination**
   - Merging bit fields
   - Combining options
   - State aggregation

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
   - Explain flag combinations
   - Maintain bit maps

## Common Patterns

### Setting Flags

```javascript
// Set multiple feature flags
db.collection.update(
  { _id: 1 },
  {
    $set: {
      features: {
        $bitor: [
          "$currentFeatures",
          NEW_FEATURE_FLAG
        ]
      }
    }
  }
)
```

### Permission Combination

```javascript
// Combine user permissions
db.users.update(
  { _id: userId },
  {
    $set: {
      permissions: {
        $bitor: [
          "$basePermissions",
          "$rolePermissions"
        ]
      }
    }
  }
)
```

## Related Operators

- [$bitand](bitand.md) - Performs bitwise AND
- [$bitnot](bitnot.md) - Performs bitwise NOT
- [$bitxnor](bitxnor.md) - Performs bitwise XNOR 