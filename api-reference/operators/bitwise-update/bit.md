# $bit

Performs bitwise update operations on integer values.

## Syntax

```javascript
{
  $bit: {
    <field>: {
      <operation>: <number>
    }
  }
}
```

Where `<operation>` can be:
- `and`: Bitwise AND operation
- `or`: Bitwise OR operation
- `xor`: Bitwise XOR operation

## Description

The `$bit` operator performs a bitwise update operation on a numeric field. It supports three types of operations:
- AND: Sets each bit to 1 only if both bits are 1
- OR: Sets each bit to 1 if either bit is 1
- XOR: Sets each bit to 1 if exactly one bit is 1

## Examples

### Using AND Operation

```javascript
// Initial document
{ _id: 1, access: 7 }  // Binary: 0111

// Remove read permission (bit 0)
db.collection.update(
  { _id: 1 },
  {
    $bit: {
      access: { and: 6 }  // Binary: 0110
    }
  }
)

// Result: { _id: 1, access: 6 }  // Binary: 0110
```

### Using OR Operation

```javascript
// Initial document
{ _id: 2, flags: 5 }  // Binary: 0101

// Add execute permission (bit 1)
db.collection.update(
  { _id: 2 },
  {
    $bit: {
      flags: { or: 2 }  // Binary: 0010
    }
  }
)

// Result: { _id: 2, flags: 7 }  // Binary: 0111
```

### Using XOR Operation

```javascript
// Initial document
{ _id: 3, status: 10 }  // Binary: 1010

// Toggle bits 1 and 3
db.collection.update(
  { _id: 3 },
  {
    $bit: {
      status: { xor: 10 }  // Binary: 1010
    }
  }
)

// Result: { _id: 3, status: 0 }  // Binary: 0000
```

## Behavior

1. **Input Requirements**
   - Field must be an integer
   - Operation value must be an integer
   - Field is created if it doesn't exist

2. **Operation Types**
   - AND: Clears bits (sets to 0)
   - OR: Sets bits (sets to 1)
   - XOR: Toggles bits

3. **Special Cases**
   - Non-existent fields
   - Non-integer values
   - Null values

## Use Cases

1. **Permission Management**
   ```javascript
   // Remove multiple permissions
   db.users.update(
     { _id: userId },
     {
       $bit: {
         permissions: {
           and: ~(READ_PERMISSION | WRITE_PERMISSION)
         }
       }
     }
   )
   ```

2. **Feature Flags**
   ```javascript
   // Enable multiple features
   db.config.update(
     { _id: configId },
     {
       $bit: {
         features: {
           or: FEATURE_A | FEATURE_B
         }
       }
     }
   )
   ```

3. **State Management**
   ```javascript
   // Toggle specific states
   db.devices.update(
     { _id: deviceId },
     {
       $bit: {
         state: {
           xor: STATE_BITS
         }
       }
     }
   )
   ```

## Performance Considerations

1. **Atomic Operations**
   - Operations are atomic
   - No read-modify-write cycle
   - Thread-safe updates

2. **Index Impact**
   - Updates affect indexes
   - Consider index maintenance
   - Plan for write amplification

3. **Batch Processing**
   - Can be used in bulk operations
   - Efficient for multiple documents
   - Consider batching related updates

## Best Practices

1. **Operation Selection**
   - Use AND to clear bits
   - Use OR to set bits
   - Use XOR to toggle bits

2. **Error Handling**
   - Validate input values
   - Handle type conversions
   - Check field existence

3. **Documentation**
   - Document bit meanings
   - Track bit assignments
   - Maintain bit maps

## Common Patterns

### Permission Masking

```javascript
// Clear specific permission bits
db.users.update(
  { _id: userId },
  {
    $bit: {
      permissions: {
        and: PERMISSION_MASK
      }
    }
  }
)
```

### Feature Activation

```javascript
// Set multiple feature bits
db.settings.update(
  { _id: settingId },
  {
    $bit: {
      features: {
        or: FEATURE_BITS
      }
    }
  }
)
```

### State Toggle

```javascript
// Toggle state bits
db.status.update(
  { _id: statusId },
  {
    $bit: {
      flags: {
        xor: TOGGLE_MASK
      }
    }
  }
)
``` 