# Bitwise Update Operators

Bitwise update operators perform bit-level operations to modify values in documents. These operators allow you to manipulate individual bits within numeric fields during update operations.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$bit](bit.md) | Performs bitwise update operations on integer values |

## Common Use Cases

- Permission management
- Feature flag updates
- State modifications
- Hardware control
- Configuration updates
- Binary data manipulation

## Example Usage

```javascript
// Set specific bits using AND operation
db.collection.update(
  { _id: 1 },
  {
    $bit: {
      permissions: { and: 14 }  // Binary: 1110
    }
  }
)

// Set specific bits using OR operation
db.collection.update(
  { _id: 2 },
  {
    $bit: {
      flags: { or: 3 }  // Binary: 0011
    }
  }
)

// Set specific bits using XOR operation
db.collection.update(
  { _id: 3 },
  {
    $bit: {
      status: { xor: 5 }  // Binary: 0101
    }
  }
)
```

## Best Practices

1. **Operation Selection**
   - Choose appropriate bitwise operation
   - Consider operation order
   - Document bit changes

2. **Performance**
   - Use atomic operations
   - Consider index impact
   - Batch updates when possible

3. **Data Integrity**
   - Validate input values
   - Handle edge cases
   - Document bit meanings

## Operator Details

### $bit
- Supports AND, OR, and XOR operations
- Operates on integer fields
- Performs atomic updates
- Useful for flag manipulation

## Common Patterns

### Permission Updates

```javascript
// Remove specific permissions using AND
db.users.update(
  { _id: userId },
  {
    $bit: {
      permissions: {
        and: ~REVOKED_PERMISSIONS  // Inverted bits of permissions to revoke
      }
    }
  }
)
```

### Feature Flag Management

```javascript
// Enable features using OR
db.config.update(
  { _id: configId },
  {
    $bit: {
      features: {
        or: NEW_FEATURES  // Bits representing new features
      }
    }
  }
)
```

### State Toggle

```javascript
// Toggle states using XOR
db.devices.update(
  { _id: deviceId },
  {
    $bit: {
      status: {
        xor: TOGGLE_BITS  // Bits to toggle
      }
    }
  }
)
``` 