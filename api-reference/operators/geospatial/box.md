# $box

The `$box` operator specifies a rectangular box for a geospatial query using legacy coordinate pairs.

## Syntax

```javascript
{
  <location field>: {
    $geoWithin: {
      $box: [ [ <bottom left coordinates> ], [ <upper right coordinates> ] ]
    }
  }
}
```

## Description

The `$box` operator:
- Defines a rectangle for geospatial queries
- Uses two coordinate pairs to specify opposite corners
- Must be used within a `$geoWithin` operator
- Works with legacy coordinate pairs [longitude, latitude]

## Examples

### Basic Box Query

```javascript
// Find locations within a rectangular area
db.places.find({
   location: {
      $geoWithin: {
         $box: [
            [0, 0],    // Bottom left corner
            [100, 100] // Upper right corner
         ]
      }
   }
})
```

### Box Query with Specific Area

```javascript
// Find restaurants in downtown area
db.restaurants.find({
   coordinates: {
      $geoWithin: {
         $box: [
            [-74.0, 40.7],  // SW corner of downtown
            [-73.9, 40.8]   // NE corner of downtown
         ]
      }
   }
})
```

### Combining with Other Criteria

```javascript
// Find active stores within a delivery area
db.stores.find({
   status: "active",
   location: {
      $geoWithin: {
         $box: [
            [-122.5, 37.7],
            [-122.4, 37.8]
         ]
      }
   }
})
```

## Behavior

- Coordinates must be specified in [longitude, latitude] order
- The rectangle is defined on a flat coordinate system
- Does not consider the curvature of the Earth
- For spherical calculations, use `$geometry` with GeoJSON instead

## Performance Considerations

- Requires a 2d index on the location field
- More efficient than polygon queries for rectangular areas
- Consider using GeoJSON and `$geometry` for more accurate Earth-surface queries
- Index usage is optimal when the box sides align with coordinate axes

## Index Requirements

```javascript
// Create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Maximum longitude: ±180
- Maximum latitude: ±90
- Cannot cross the 180th meridian
- Not suitable for spherical calculations
- Legacy coordinate pairs only (not GeoJSON)

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$geometry](geometry.md) - Specifies a GeoJSON geometry
- [$polygon](polygon.md) - Defines a polygon for geospatial queries
- [$center](center.md) - Defines a circle for geospatial queries 