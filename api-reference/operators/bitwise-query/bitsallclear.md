# $bitsAllClear

Matches numeric or binary values where all specified bit positions have a value of 0.

## Syntax

```javascript
{ <field>: { $bitsAllClear: <bitmask> } }
```

The `<bitmask>` can be either:
- An array of bit positions
- A number representing the bitmask
- A BinData object

## Description

The `$bitsAllClear` operator matches documents where all the bit positions specified in the `<bitmask>` have a value of 0 in the field. This is useful for checking if multiple flags are all disabled or if specific bits in a binary field are all clear.

## Examples

### Using Bit Positions Array

```javascript
// Initial documents
{ _id: 1, permissions: 10 }  // Binary: 1010
{ _id: 2, permissions: 20 }  // Binary: 10100
{ _id: 3, permissions: 15 }  // Binary: 1111

// Find documents where bits 1 and 3 are both clear (0)
db.collection.find({
  permissions: {
    $bitsAllClear: [1, 3]
  }
})

// Returns: { _id: 2, permissions: 20 }
```

### Using Numeric Bitmask

```javascript
// Initial document
{ _id: 4, flags: 12 }  // Binary: 1100

// Check if bits corresponding to mask 3 (Binary: 11) are clear
db.collection.find({
  flags: {
    $bitsAllClear: 3
  }
})

// No match because at least one of the bits is set
```

### Using with Binary Data

```javascript
// Initial document
{
  _id: 5,
  data: BinData(0, "AQIDBA==")  // Binary data
}

// Check specific bits in binary data
db.collection.find({
  data: {
    $bitsAllClear: BinData(0, "Cg==")
  }
})
```

## Behavior

1. **Input Types**
   - Integer values
   - Long values
   - Binary data (BinData)
   - Decimal128 (converted to long)

2. **Bitmask Formats**
   - Array of bit positions (0-based)
   - Numeric bitmask
   - BinData object

3. **Special Cases**
   - Null values
   - Missing fields
   - Invalid bit positions
   - Out of range positions

## Use Cases

1. **Permission Checking**
   ```javascript
   // Check if user has no admin privileges
   db.users.find({
     privileges: {
       $bitsAllClear: [
         ADMIN_BIT,
         SUPER_USER_BIT
       ]
     }
   })
   ```

2. **Feature Management**
   ```javascript
   // Find disabled features
   db.config.find({
     features: {
       $bitsAllClear: FEATURE_MASK
     }
   })
   ```

3. **Hardware Status**
   ```javascript
   // Check for inactive components
   db.devices.find({
     status: {
       $bitsAllClear: [
         POWER_BIT,
         NETWORK_BIT,
         ERROR_BIT
       ]
     }
   })
   ```

## Performance Considerations

1. **Index Usage**
   - Can use indexes on the queried field
   - Consider selectivity of the query
   - May benefit from compound indexes

2. **Query Optimization**
   - Use appropriate bitmask format
   - Consider bit position ordering
   - Combine with other query conditions

3. **Memory Usage**
   - Efficient for large datasets
   - Minimal memory overhead
   - Constant time operation

## Best Practices

1. **Input Validation**
   - Validate bit positions
   - Check range limits
   - Handle edge cases

2. **Error Handling**
   - Handle null values
   - Check field existence
   - Validate input types

3. **Documentation**
   - Document bit meanings
   - Maintain bit maps
   - Track bit assignments

## Related Operators

- [$bitsAllSet](bitsallset.md) - Matches when all specified bits are 1
- [$bitsAnyClear](bitsanyclear.md) - Matches when any specified bit is 0
- [$bitsAnySet](bitsanyset.md) - Matches when any specified bit is 1 