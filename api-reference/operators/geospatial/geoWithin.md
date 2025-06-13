# $geoWithin

The `$geoWithin` operator selects documents with geospatial data that exists entirely within a specified shape.

## Syntax

```javascript
{
  <location field>: {
    $geoWithin: {
      <geometry operator>: <shape specification>
    }
  }
}
```

## Description

The `$geoWithin` operator:
- Selects documents whose geospatial data falls entirely within a specified shape
- Works with both GeoJSON geometries and legacy shape operators
- Does not require a geospatial index, but benefits from one
- Supports multiple shape specifications
- Does not sort results by distance

## Examples

### Using GeoJSON Polygon

```javascript
// Find locations within a polygon
db.places.find({
   location: {
      $geoWithin: {
         $geometry: {
            type: "Polygon",
            coordinates: [[
               [-74.0, 40.7],
               [-73.9, 40.7],
               [-73.9, 40.8],
               [-74.0, 40.8],
               [-74.0, 40.7]
            ]]
         }
      }
   }
})
```

### Using Legacy Circle

```javascript
// Find locations within a circular area
db.places.find({
   location: {
      $geoWithin: {
         $center: [
            [-73.93, 40.82],  // Center point
            5                 // Radius
         ]
      }
   }
})
```

### Using Legacy Box

```javascript
// Find locations within a rectangular area
db.places.find({
   location: {
      $geoWithin: {
         $box: [
            [-122.4, 37.7],   // Bottom left corner
            [-122.3, 37.8]    // Upper right corner
         ]
      }
   }
})
```

## Behavior

- Documents must fall completely within the specified shape
- Supports multiple geometry operators:
  - `$geometry` - For GeoJSON shapes
  - `$box` - For rectangular shapes
  - `$center` - For circular shapes on flat surfaces
  - `$centerSphere` - For circular shapes on spherical surfaces
  - `$polygon` - For arbitrary polygons
- Results are not sorted by distance
- Works with both 2d and 2dsphere indexes

## Performance Considerations

- Benefits from geospatial indexes but doesn't require them
- More efficient than `$geoIntersects` for containment queries
- Performance varies by shape complexity
- Index usage depends on the geometry operator used
- Consider using simpler shapes for better performance

## Index Requirements

```javascript
// Create a 2dsphere index for GeoJSON data
db.collection.createIndex({ location: "2dsphere" })

// Or create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Maximum document size limits apply to shapes
- Legacy operators have coordinate limitations
- GeoJSON polygons must be closed (first and last points must match)
- Cannot sort results by distance
- Some operators only work with specific index types

## Related Operators

- [$geometry](geometry.md) - Specifies a GeoJSON geometry
- [$box](box.md) - Defines a rectangle for geospatial queries
- [$center](center.md) - Defines a circle for geospatial queries
- [$centerSphere](centerSphere.md) - Defines a circle on a sphere
- [$polygon](polygon.md) - Defines a polygon for geospatial queries 