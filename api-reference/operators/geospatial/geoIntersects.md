# $geoIntersects

The `$geoIntersects` operator selects documents whose geospatial data intersects with a specified GeoJSON geometry.

## Syntax

```javascript
{
  <location field>: {
    $geoIntersects: {
      $geometry: {
        type: <GeoJSON type>,
        coordinates: <coordinates>
      }
    }
  }
}
```

## Description

The `$geoIntersects` operator:
- Selects documents that intersect with a specified GeoJSON geometry
- Supports all GeoJSON geometry types
- Requires a 2dsphere index for optimal performance
- Works only with GeoJSON objects (not legacy coordinates)
- Returns documents where the geometry intersects at any point

## Examples

### Basic Intersection Query

```javascript
// Find all locations that intersect with a specific line
db.places.find({
   location: {
      $geoIntersects: {
         $geometry: {
            type: "LineString",
            coordinates: [
               [-73.93, 40.82],
               [-73.94, 40.81]
            ]
         }
      }
   }
})
```

### Polygon Intersection

```javascript
// Find all routes that intersect with a polygon
db.routes.find({
   path: {
      $geoIntersects: {
         $geometry: {
            type: "Polygon",
            coordinates: [[
               [-122.4, 37.7],
               [-122.4, 37.8],
               [-122.5, 37.8],
               [-122.5, 37.7],
               [-122.4, 37.7]
            ]]
         }
      }
   }
})
```

### Complex Geometry Intersection

```javascript
// Find features that intersect with a MultiPolygon
db.features.find({
   geometry: {
      $geoIntersects: {
         $geometry: {
            type: "MultiPolygon",
            coordinates: [
               [[[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]]],
               [[[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]]]
            ]
         }
      }
   }
})
```

## Behavior

- Returns documents where geometries intersect
- Supports all GeoJSON geometry types:
  - Point
  - LineString
  - Polygon
  - MultiPoint
  - MultiLineString
  - MultiPolygon
  - GeometryCollection
- Does not consider the interior of polygons
- Intersection is determined by shared points or edges

## Performance Considerations

- Requires a 2dsphere index
- Performance depends on the complexity of geometries
- More efficient with simpler geometries
- Consider breaking complex geometries into simpler ones
- Index usage is optimal for smaller geometries

## Index Requirements

```javascript
// Create a 2dsphere index
db.collection.createIndex({ location: "2dsphere" })
```

## Limitations

- Only works with GeoJSON objects
- Does not support legacy coordinate pairs
- Maximum document size limits apply to geometries
- Complex geometries may impact performance
- Cannot use with spherical queries

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$geometry](geometry.md) - Specifies a GeoJSON geometry
- [$near](near.md) - Selects points near a point
- [$box](box.md) - Defines a rectangle for geospatial queries 