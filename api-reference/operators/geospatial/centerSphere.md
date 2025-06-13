# $centerSphere

The `$centerSphere` operator specifies a circle for a geospatial query using spherical geometry.

## Syntax

```javascript
{
  <location field>: {
    $geoWithin: {
      $centerSphere: [ [ <longitude>, <latitude> ], <radius in radians> ]
    }
  }
}
```

## Description

The `$centerSphere` operator:
- Defines a circle for geospatial queries using spherical geometry
- Uses a center point and radius in radians
- Must be used within a `$geoWithin` operator
- Works with both legacy coordinates and GeoJSON
- Calculates distances along a sphere (ideal for Earth-surface calculations)

## Examples

### Basic Spherical Query

```javascript
// Find locations within 5 miles of a point
db.places.find({
   location: {
      $geoWithin: {
         $centerSphere: [
            [-73.93, 40.82],  // [longitude, latitude]
            5 / 3963.2        // 5 miles radius (converted to radians)
         ]
      }
   }
})
```

### Finding Nearby Restaurants

```javascript
// Find restaurants within 10 kilometers
db.restaurants.find({
   coordinates: {
      $geoWithin: {
         $centerSphere: [
            [-122.27, 37.80],  // San Francisco coordinates
            10 / 6371.0        // 10 km radius (converted to radians)
         ]
      }
   }
})
```

### Combining with Other Criteria

```javascript
// Find active stores within delivery radius
db.stores.find({
   status: "active",
   location: {
      $geoWithin: {
         $centerSphere: [
            [-0.127, 51.507],  // London coordinates
            2 / 6371.0         // 2 km delivery radius
         ]
      }
   },
   isOpen: true
})
```

## Behavior

- Coordinates must be specified in [longitude, latitude] order
- Radius must be specified in radians
- Common radius conversions:
  - Miles to radians: divide by 3963.2
  - Kilometers to radians: divide by 6371.0
- Calculations consider Earth's curvature
- More accurate for Earth-surface calculations than `$center`

## Performance Considerations

- Requires a 2dsphere or 2d index on the location field
- More accurate than `$center` for Earth-surface queries
- Optimal for geographic calculations
- Index usage is efficient for both small and large radius values

## Index Requirements

```javascript
// Create a 2dsphere index for GeoJSON data
db.collection.createIndex({ location: "2dsphere" })

// Or create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Maximum longitude: ±180
- Maximum latitude: ±90
- Radius must be positive and less than π radians
- Performance may degrade with very large radius values
- Legacy coordinates must be in [longitude, latitude] order

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$center](center.md) - Defines a circle for flat geometry calculations
- [$near](near.md) - Selects points near a point
- [$nearSphere](nearSphere.md) - Selects points near a point on a sphere 