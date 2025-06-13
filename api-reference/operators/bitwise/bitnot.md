# $bitnot

Performs a bitwise NOT operation on a number.

## Syntax

```javascript
{ <field>: { $bitnot: <number> } }
```

## Description

The `$bitnot` operator performs a bitwise NOT operation on an integer. It inverts each bit of the operand, changing 1s to 0s and 0s to 1s. The result is the one's complement of the input value.

## Examples

### Basic Bitwise NOT

```javascript
// Initial document
{ _id: 1, value: 5 }  // 5 in binary is 0101

// Find documents where inverted value matches
db.collection.find({
  value: {
    $bitnot: 5  // Result: -6 (1010 in two's complement)
  }
})
```

### Mask Inversion

```javascript
// Initial document
{
  _id: 2,
  mask: 15  // 1111 in binary
}

// Create inverted mask
db.collection.update(
  { _id: 2 },
  {
    $set: {
      invertedMask: {
        $bitnot: "$mask"
      }
    }
  }
)
```

### Bit Pattern Matching

```javascript
// Initial document
{
  _id: 3,
  pattern: 170  // 10101010 in binary
}

// Match inverse pattern
db.collection.find({
  pattern: {
    $bitnot: 85  // 01010101 in binary
  }
})
```

## Behavior

1. **Input Requirements**
   - Accepts integer values
   - Converts to 32-bit integer
   - Handles negative numbers

2. **Result Type**
   - Returns 32-bit integer
   - Uses two's complement
   - Preserves sign bit

3. **Special Cases**
   - Null or missing fields
   - Non-integer values
   - Boundary values

## Use Cases

1. **Bit Manipulation**
   - Creating bit masks
   - Pattern inversion
   - Binary operations

2. **Data Processing**
   - Signal processing
   - Image processing
   - Data encoding

3. **Logic Operations**
   - Boolean complement
   - State inversion
   - Flag toggling

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
   - Explain inversions
   - Maintain bit maps

## Common Patterns

### Creating Masks

```javascript
// Create inverse mask for bit operations
db.collection.update(
  { _id: 1 },
  {
    $set: {
      mask: {
        $bitnot: existingPattern
      }
    }
  }
)
```

### Pattern Matching

```javascript
// Find documents with complementary patterns
db.collection.find({
  pattern: {
    $bitnot: targetPattern
  }
})
```

## Related Operators

- [$bitand](bitand.md) - Performs bitwise AND
- [$bitor](bitor.md) - Performs bitwise OR
- [$bitxnor](bitxnor.md) - Performs bitwise XNOR 