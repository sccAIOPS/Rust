# Navigation Algorithms

This directory contains comprehensive documentation for geographic navigation algorithms used in calculating distances, bearings, and positions on Earth's surface.

## Algorithms

| Algorithm | File | Description | Complexity |
|-----------|------|-------------|------------|
| Haversine Formula | [haversine.md](haversine.md) | Great circle distance between two points | O(1) |
| Bearing Calculation | [bearing.md](bearing.md) | Initial bearing between two points | O(1) |
| Rhumb Line Distance | [rhumbline.md](rhumbline.md) | Distance along rhumb line (constant bearing) | O(1) |

## Overview

Navigation algorithms are fundamental to geographic information systems (GIS), mapping applications, aviation, maritime navigation, and location-based services. These algorithms work with:

- **Latitude/Longitude**: Geographic coordinates on Earth's surface
- **Great Circle**: Shortest path between two points on a sphere
- **Rhumb Line**: Path with constant bearing (not shortest but easier to navigate)
- **Bearing**: Direction from one point to another

## Common Applications

1. **GPS Navigation Systems**: Route planning and distance calculation
2. **Aviation**: Flight path planning and navigation
3. **Maritime Navigation**: Ship routing and position tracking
4. **Mapping Applications**: Distance measurement and direction finding
5. **Location-Based Services**: Proximity search and geofencing
6. **Geographic Analysis**: Spatial data analysis and clustering

## Coordinate Systems

All algorithms assume:
- **Latitude**: -90° (South Pole) to +90° (North Pole)
- **Longitude**: -180° (West) to +180° (East)
- **Earth Model**: Spherical approximation (radius ≈ 6,371 km)

## Mathematical Foundation

### Spherical Earth Model

These algorithms use a spherical Earth approximation with:
- Mean Earth radius: 6,371,000 meters
- Angular conversions: radians ↔ degrees

While more accurate models exist (WGS84 ellipsoid), the spherical model provides:
- Sufficient accuracy for most applications (error < 0.5%)
- Simple computations
- Fast calculation

### Coordinate Conversion

$$\text{radians} = \text{degrees} \times \frac{\pi}{180}$$

$$\text{degrees} = \text{radians} \times \frac{180}{\pi}$$

## References

1. Sinnott, R. W. (1984). "Virtues of the Haversine". Sky and Telescope, 68(2), 159.
2. Vincenty, T. (1975). "Direct and Inverse Solutions of Geodesics on the Ellipsoid"
3. Williams, E. (n.d.). "Aviation Formulary V1.47"
4. Movable Type Scripts: [www.movable-type.co.uk/scripts/latlong.html](https://www.movable-type.co.uk/scripts/latlong.html)

## Implementation Notes

### Rust-Specific Considerations

- Uses `f64` for high precision floating-point calculations
- Leverages `std::f64::consts::PI` for mathematical constants
- Handles edge cases like antimeridian crossing
- Returns distances in meters (SI units)
- Returns bearings in degrees (0-360°)

### Accuracy Considerations

- Spherical model error increases near poles
- For high-precision applications, consider ellipsoidal models
- Floating-point precision limits accuracy to ~1 micrometer
- Bearing calculations can be unstable for very short distances
