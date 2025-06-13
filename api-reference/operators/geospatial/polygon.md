# $polygon

The `$polygon` operator specifies a polygon for a geospatial query using legacy coordinate pairs.

## Syntax

```javascript
{
  <location field>: {
    $geoWithin: {
      $polygon: [ [ <x1>, <y1> ], [ <x2>, <y2> ], [ <x3>, <y3> ], ..., [ <xn>, <yn> ] ]
    }
  }
}
```

## Description

The `$polygon` operator:
- Defines a polygon for geospatial queries using legacy coordinates
- Must be used within a `$geoWithin` operator
- Works with legacy coordinate pairs [x, y]
- Requires at least three points to form a polygon
- Points must be specified in order to form a valid polygon

## Examples

### Basic Polygon Query

```javascript
// Find locations within a triangular area
db.places.find({
   location: {
      $geoWithin: {
         $polygon: [
            [0, 0],
            [2, 2],
            [0, 2],
            [0, 0]  // Close the polygon
         ]
      }
   }
})
```

### Complex Polygon Query

```javascript
// Find locations within a quadrilateral
db.places.find({
   location: {
      $geoWithin: {
         $polygon: [
            [-73.93, 40.82],
            [-73.93, 40.85],
            [-73.96, 40.85],
            [-73.96, 40.82],
            [-73.93, 40.82]  // Close the polygon
         ]
      }
   }
})
```

### With Additional Criteria

```javascript
// Find active stores within a polygon area
db.stores.find({
   location: {
      $geoWithin: {
         $polygon: [
            [-122.4, 37.7],
            [-122.4, 37.8],
            [-122.5, 37.8],
            [-122.5, 37.7],
            [-122.4, 37.7]  // Close the polygon
         ]
      }
   },
   status: "active",
   type: "retail"
})
```

## Behavior

- Points must be specified in [x, y] order
- Polygon must be closed (first and last points must match)
- Minimum of three points required
- Points must be specified in order to form a valid polygon
- Uses flat geometry calculations (not spherical)
- Does not consider Earth's curvature

## Performance Considerations

- Requires a 2d index on the location field
- Performance depends on polygon complexity
- Simpler polygons provide better performance
- Consider breaking complex polygons into simpler ones
- Index usage is optimal for convex polygons

## Index Requirements

```javascript
// Create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Only works with legacy coordinate pairs
- Not suitable for spherical calculations
- Maximum of 100,000 points per polygon
- Cannot cross the 180th meridian
- Must be a closed polygon
- Points must be in correct order

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$box](box.md) - Defines a rectangle for geospatial queries
- [$center](center.md) - Defines a circle for geospatial queries
- [$geometry](geometry.md) - Specifies a GeoJSON geometry 