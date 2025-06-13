# $bitsAllSet

Matches numeric or binary values where all specified bit positions have a value of 1.

## Syntax

```javascript
{ <field>: { $bitsAllSet: <bitmask> } }
```

The `<bitmask>` can be either:
- An array of bit positions
- A number representing the bitmask
- A BinData object

## Description

The `$bitsAllSet` operator matches documents where all the bit positions specified in the `<bitmask>` have a value of 1 in the field. This is useful for checking if multiple flags are all enabled or if specific bits in a binary field are all set.

## Examples

### Using Bit Positions Array

```javascript
// Initial documents
{ _id: 1, permissions: 11 }  // Binary: 1011
{ _id: 2, permissions: 7 }   // Binary: 0111
{ _id: 3, permissions: 15 }  // Binary: 1111

// Find documents where bits 0 and 1 are both set (1)
db.collection.find({
  permissions: {
    $bitsAllSet: [0, 1]
  }
})

// Returns: { _id: 1, permissions: 11 }, { _id: 2, permissions: 7 }, { _id: 3, permissions: 15 }
```

### Using Numeric Bitmask

```javascript
// Initial document
{ _id: 4, flags: 12 }  // Binary: 1100

// Check if bits corresponding to mask 12 (Binary: 1100) are set
db.collection.find({
  flags: {
    $bitsAllSet: 12
  }
})

// Returns: { _id: 4, flags: 12 }
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
    $bitsAllSet: BinData(0, "AQ==")
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

1. **Permission Verification**
   ```javascript
   // Check if user has all required permissions
   db.users.find({
     privileges: {
       $bitsAllSet: [
         READ_BIT,
         WRITE_BIT,
         EXECUTE_BIT
       ]
     }
   })
   ```

2. **Feature Activation**
   ```javascript
   // Find fully enabled features
   db.config.find({
     features: {
       $bitsAllSet: REQUIRED_FEATURES
     }
   })
   ```

3. **Status Monitoring**
   ```javascript
   // Check for active components
   db.devices.find({
     status: {
       $bitsAllSet: [
         ONLINE_BIT,
         READY_BIT,
         HEALTHY_BIT
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

## Common Patterns

### Role-Based Access Control

```javascript
// Check for admin privileges
db.users.find({
  roles: {
    $bitsAllSet: ADMIN_PRIVILEGES
  }
})
```

### Configuration Validation

```javascript
// Verify required settings
db.settings.find({
  config: {
    $bitsAllSet: MANDATORY_CONFIG
  }
})
```

## Related Operators

- [$bitsAllClear](bitsallclear.md) - Matches when all specified bits are 0
- [$bitsAnyClear](bitsanyclear.md) - Matches when any specified bit is 0
- [$bitsAnySet](bitsanyset.md) - Matches when any specified bit is 1 