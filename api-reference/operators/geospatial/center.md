# $center

The `$center` operator specifies a circle using legacy coordinate pairs for a geospatial query.

## Syntax

```javascript
{
  <location field>: {
    $geoWithin: {
      $center: [ [ <x>, <y> ], <radius> ]
    }
  }
}
```

## Description

The `$center` operator:
- Defines a circle for geospatial queries
- Uses a center point and radius to specify the circle
- Must be used within a `$geoWithin` operator
- Works with legacy coordinate pairs [x, y]
- Uses flat geometry calculations (not spherical)

## Examples

### Basic Circle Query

```javascript
// Find locations within a circular area
db.places.find({
   location: {
      $geoWithin: {
         $center: [
            [0, 0],  // Center point [x, y]
            10       // Radius
         ]
      }
   }
})
```

### Circle Query with Specific Area

```javascript
// Find restaurants within 5 units of a point
db.restaurants.find({
   coordinates: {
      $geoWithin: {
         $center: [
            [-73.93, 40.82],  // Center point
            5                 // Radius
         ]
      }
   }
})
```

### Combining with Other Criteria

```javascript
// Find active stores within a circular delivery area
db.stores.find({
   status: "active",
   location: {
      $geoWithin: {
         $center: [
            [-122.45, 37.75],  // Center of delivery area
            5                  // Delivery radius
         ]
      }
   }
})
```

## Behavior

- Coordinates must be specified in [x, y] order
- The circle is defined on a flat coordinate system
- Does not consider the curvature of the Earth
- For spherical calculations, use `$centerSphere` instead
- Radius is specified in the same units as the coordinates

## Performance Considerations

- Requires a 2d index on the location field
- More efficient than polygon queries for circular areas
- Consider using `$centerSphere` for Earth-surface queries
- Index usage is optimal for smaller radius values

## Index Requirements

```javascript
// Create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Not suitable for spherical calculations
- Legacy coordinate pairs only (not GeoJSON)
- Radius must be positive
- Performance degrades with very large radius values

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$centerSphere](centerSphere.md) - Defines a circle for spherical calculations
- [$box](box.md) - Defines a rectangle for geospatial queries
- [$polygon](polygon.md) - Defines a polygon for geospatial queries 