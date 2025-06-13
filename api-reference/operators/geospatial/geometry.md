# $geometry

The `$geometry` operator specifies a GeoJSON geometry for geospatial queries.

## Syntax

```javascript
{
  $geometry: {
    type: <GeoJSON type>,
    coordinates: <coordinates>
  }
}
```

## Description

The `$geometry` operator:
- Specifies a geometry in GeoJSON format
- Used within other geospatial operators
- Supports all GeoJSON geometry types
- Works with 2dsphere indexes
- Allows for precise geospatial queries

## Examples

### Point Geometry

```javascript
// Find locations near a point
db.places.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-73.93, 40.82]
         },
         $maxDistance: 1000
      }
   }
})
```

### Polygon Geometry

```javascript
// Find locations within a polygon
db.places.find({
   location: {
      $geoWithin: {
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

### MultiPoint Geometry

```javascript
// Find locations that intersect with multiple points
db.places.find({
   location: {
      $geoIntersects: {
         $geometry: {
            type: "MultiPoint",
            coordinates: [
               [-73.93, 40.82],
               [-73.94, 40.81],
               [-73.92, 40.83]
            ]
         }
      }
   }
})
```

## Behavior

- Requires valid GeoJSON format
- Supports all GeoJSON geometry types:
  - Point
  - LineString
  - Polygon
  - MultiPoint
  - MultiLineString
  - MultiPolygon
  - GeometryCollection
- Coordinates must be in [longitude, latitude] order
- Polygons must be closed (first and last points must match)
- Can include additional GeoJSON properties

## Performance Considerations

- Works best with 2dsphere indexes
- Performance depends on geometry complexity
- Simpler geometries provide better performance
- Consider breaking complex geometries into simpler ones
- Index usage is optimal for common geometry types

## Index Requirements

```javascript
// Create a 2dsphere index
db.collection.createIndex({ location: "2dsphere" })
```

## Limitations

- Only works with GeoJSON format
- Maximum document size limits apply
- Complex geometries may impact performance
- Must follow GeoJSON specification strictly
- Coordinates must be valid longitude/latitude values

## Related Operators

- [$geoWithin](geoWithin.md) - Selects geometries within a bounding geometry
- [$geoIntersects](geoIntersects.md) - Selects geometries that intersect
- [$near](near.md) - Selects points near a point
- [$nearSphere](nearSphere.md) - Selects points near a point on a sphere 