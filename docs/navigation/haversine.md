# Haversine Formula

## 1. Overview

The **Haversine formula** calculates the great-circle distance between two points on a sphere given their latitudes and longitudes. It is named after the haversine function:

$$\text{haversin}(\theta) = \sin^2\left(\frac{\theta}{2}\right) = \frac{1 - \cos(\theta)}{2}$$

Invented by James Inman in 1835, this formula was crucial for celestial navigation before computers. The haversine function was chosen because it remains numerically stable for small distances, avoiding the catastrophic cancellation that can occur with the law of cosines for small angles.

The algorithm computes the shortest distance over Earth's surface, following the curvature of the planet (great circle path), making it essential for navigation systems, geographic information systems, and location-based services.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: Two points on Earth's surface
- Point 1: $(lat_1, lng_1)$ in degrees
- Point 2: $(lat_2, lng_2)$ in degrees

**Output**: Great-circle distance $d$ in meters

**Earth Model**: Spherical approximation with radius $R = 6,371,000$ meters

### 2.2 Mathematical Model

The haversine formula for the central angle $\theta$ between two points on a sphere:

$$a = \sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1) \cdot \cos(\phi_2) \cdot \sin^2\left(\frac{\Delta\lambda}{2}\right)$$

$$c = 2 \cdot \arcsin(\sqrt{a})$$

$$d = R \cdot c$$

Where:
- $\phi_1, \phi_2$ = latitude of point 1 and 2 (in radians)
- $\lambda_1, \lambda_2$ = longitude of point 1 and 2 (in radians)
- $\Delta\phi = \phi_2 - \phi_1$ (latitude difference)
- $\Delta\lambda = \lambda_2 - \lambda_1$ (longitude difference)
- $R$ = Earth's mean radius (6,371 km)
- $d$ = great-circle distance

### 2.3 Derivation

The formula derives from the spherical law of cosines:

$$d = \arccos(\sin(\phi_1) \cdot \sin(\phi_2) + \cos(\phi_1) \cdot \cos(\phi_2) \cdot \cos(\Delta\lambda)) \cdot R$$

However, this suffers from rounding errors for small distances (when $d$ is small, $\arccos$ approaches 1, losing precision). The haversine reformulation uses:

$$\sin^2\left(\frac{\theta}{2}\right) = \frac{1 - \cos(\theta)}{2}$$

This maintains numerical stability by avoiding subtraction of similar quantities.

### 2.4 Accuracy Considerations

- **Spherical Model Error**: ±0.5% (Earth is slightly ellipsoidal)
- **Numerical Precision**: ~10⁻¹² radians (~0.006 meters)
- **Valid Range**: Any two points on Earth
- **Edge Cases**: 
  - Antipodal points (opposite sides of Earth): distance = πR
  - Same point: distance = 0
  - Points on equator: simplified calculation possible

## 3. Algorithm Description

### 3.1 Intuition

Imagine drawing a circle on Earth's surface that:
1. Passes through both your starting and ending points
2. Has its center at Earth's center
3. Divides Earth into two equal hemispheres

This is a "great circle." The shortest path between any two points on a sphere follows this great circle (like how airplanes fly). The haversine formula calculates this shortest distance by:

1. Converting latitude/longitude from degrees to radians
2. Computing the differences in latitude and longitude
3. Using the haversine function to find the central angle
4. Multiplying the angle by Earth's radius to get the distance

### 3.2 Pseudocode

```
FUNCTION Haversine(lat1, lng1, lat2, lng2):
    // Constants
    EARTH_RADIUS = 6,371,000  // meters
    PI = 3.14159...
    
    // Step 1: Convert degrees to radians
    φ1 = lat1 × π / 180
    φ2 = lat2 × π / 180
    Δφ = (lat2 - lat1) × π / 180
    Δλ = (lng2 - lng1) × π / 180
    
    // Step 2: Calculate haversine components
    a = sin²(Δφ/2) + cos(φ1) × cos(φ2) × sin²(Δλ/2)
    
    // Step 3: Calculate central angle
    c = 2 × arcsin(√a)
    
    // Step 4: Calculate distance
    distance = EARTH_RADIUS × c
    
    RETURN distance
END FUNCTION
```

### 3.3 Step-by-Step Example

**Problem**: Calculate the distance between Amsterdam and a nearby point.

**Input**:
- Point A (Amsterdam Centraal): (52.375603°N, 4.903206°E)
- Point B (Nearby location): (52.366059°N, 4.926692°E)

**Step 1**: Convert to radians
```
φ1 = 52.375603 × π/180 = 0.914191 rad
φ2 = 52.366059 × π/180 = 0.912525 rad
Δφ = (52.366059 - 52.375603) × π/180 = -0.001666 rad
Δλ = (4.926692 - 4.903206) × π/180 = 0.000410 rad
```

**Step 2**: Calculate haversine components
```
sin²(Δφ/2) = sin²(-0.000833) = 6.943 × 10⁻⁷
sin²(Δλ/2) = sin²(0.000205) = 4.201 × 10⁻⁸
cos(φ1) = cos(0.914191) = 0.610347
cos(φ2) = cos(0.912525) = 0.611358

a = 6.943 × 10⁻⁷ + 0.610347 × 0.611358 × 4.201 × 10⁻⁸
a = 6.943 × 10⁻⁷ + 1.569 × 10⁻⁸
a = 7.100 × 10⁻⁷
```

**Step 3**: Calculate central angle
```
√a = 0.000843
c = 2 × arcsin(0.000843) = 0.001686 rad
```

**Step 4**: Calculate distance
```
distance = 6,371,000 × 0.001686 = 1,919 meters ≈ 1.92 km
```

## 4. Complexity Analysis

### 4.1 Time Complexity

**O(1)** - Constant time

The algorithm performs a fixed number of operations:
- 4 multiplications for degree-to-radian conversion
- 2 subtractions for differences
- 6 trigonometric function calls (sin, cos)
- 1 square root
- Several arithmetic operations

All operations are primitive and execute in constant time regardless of input values.

### 4.2 Space Complexity

**O(1)** - Constant space

The algorithm uses only a fixed number of variables:
- 4 input coordinates
- 4 intermediate angle values
- 2 haversine components
- 1 central angle
- 1 result distance

No additional memory allocation scales with input.

### 4.3 Performance Characteristics

- **Execution Time**: ~50-100 nanoseconds on modern CPUs
- **Numerical Stability**: Excellent for all distances
- **Precision**: Limited by floating-point representation (~10⁻¹² relative error)
- **Branch Prediction**: No conditional branches (except in special implementations)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::f64::consts::PI;

const EARTH_RADIUS: f64 = 6371000.00;

pub fn haversine(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    // Convert to radians
    let delta_dist_lat = (lat2 - lat1) * PI / 180.0;
    let delta_dist_lng = (lng2 - lng1) * PI / 180.0;
    let cos1 = lat1 * PI / 180.0;
    let cos2 = lat2 * PI / 180.0;
    
    // Calculate haversine
    let delta_lat = (delta_dist_lat / 2.0).sin().powf(2.0);
    let delta_lng = (delta_dist_lng / 2.0).sin().powf(2.0);
    let a = delta_lat + delta_lng * cos1.cos() * cos2.cos();
    
    // Calculate distance
    let result = 2.0 * a.asin().sqrt();
    result * EARTH_RADIUS
}
```

**Key Features**:
- Uses `f64` for double-precision floating-point (53 bits of precision)
- `std::f64::consts::PI` provides high-precision π constant
- `powf(2.0)` for squaring (alternative: `x * x` may be faster)
- Method chaining for readability: `(x / 2.0).sin().powf(2.0)`
- No heap allocations

**Type Constraints**:
- Input: Four `f64` parameters (latitude and longitude pairs)
- Output: `f64` distance in meters
- No generic types needed (algorithm is specific to spherical coordinates)

### 5.2 Edge Cases

**1. Identical Points**
```rust
haversine(52.0, 4.0, 52.0, 4.0) // Returns 0.0
```

**2. Antipodal Points** (opposite sides of Earth)
```rust
haversine(0.0, 0.0, 0.0, 180.0) // Returns π × R ≈ 20,015 km
```

**3. Points Across Date Line** (longitude wraps at ±180°)
```rust
haversine(0.0, 179.0, 0.0, -179.0) // 2° apart, not 358°
```
*Note: Current implementation doesn't handle this; requires normalization.*

**4. Poles**
```rust
haversine(90.0, 0.0, 90.0, 180.0) // Both at North Pole: 0.0
```

**5. Very Short Distances** (numerical stability test)
```rust
haversine(52.0, 4.0, 52.0001, 4.0001) // ~15 meters (stable)
```

**6. Invalid Coordinates** (no bounds checking)
```rust
haversine(200.0, 500.0, 300.0, 600.0) // Invalid but computes
```
*Production code should validate: -90 ≤ lat ≤ 90, -180 ≤ lng ≤ 180*

### 5.3 Optimization Opportunities

**1. Fast Approximation for Small Distances**
```rust
// For distances < 10 km, use equirectangular approximation (10x faster)
fn fast_distance(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let x = (lng2 - lng1) * (lat1 + lat2) / 2.0;
    let y = lat2 - lat1;
    (x * x + y * y).sqrt() * EARTH_RADIUS * PI / 180.0
}
```

**2. Vectorization (SIMD)**
```rust
// Process multiple coordinate pairs simultaneously
// Requires architecture-specific intrinsics or libraries like packed_simd
```

**3. Precomputed Constants**
```rust
const DEG_TO_RAD: f64 = PI / 180.0;
// Use: let rad = deg * DEG_TO_RAD
```

**4. Alternative Formulation**
```rust
// Use 2 * atan2(√a, √(1-a)) instead of 2 * asin(√a)
// Slightly more accurate for near-antipodal points
let c = 2.0 * a.sqrt().atan2((1.0 - a).sqrt());
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Location-Based Services (LBS)**
```rust
// Find all restaurants within 5 km radius
fn nearby_restaurants(user_lat: f64, user_lng: f64, restaurants: &[(f64, f64)]) -> Vec<usize> {
    restaurants.iter()
        .enumerate()
        .filter(|(_, &(lat, lng))| haversine(user_lat, user_lng, lat, lng) <= 5000.0)
        .map(|(idx, _)| idx)
        .collect()
}
```

**2. GPS Navigation Systems**
```rust
// Calculate total route distance
fn route_distance(waypoints: &[(f64, f64)]) -> f64 {
    waypoints.windows(2)
        .map(|w| haversine(w[0].0, w[0].1, w[1].0, w[1].1))
        .sum()
}
```

**3. Geofencing**
```rust
// Check if user is within geofence boundary
fn is_within_geofence(user_lat: f64, user_lng: f64, center_lat: f64, center_lng: f64, radius: f64) -> bool {
    haversine(user_lat, user_lng, center_lat, center_lng) <= radius
}
```

**4. Ride-Sharing / Delivery Apps**
```rust
// Match driver to nearest customer
fn find_nearest_driver(customer: (f64, f64), drivers: &[(f64, f64)]) -> Option<usize> {
    drivers.iter()
        .enumerate()
        .min_by(|(_, &(lat1, lng1)), (_, &(lat2, lng2))| {
            haversine(customer.0, customer.1, lat1, lng1)
                .partial_cmp(&haversine(customer.0, customer.1, lat2, lng2))
                .unwrap()
        })
        .map(|(idx, _)| idx)
}
```

**5. Aviation Flight Planning**
```rust
// Calculate fuel required based on distance
fn calculate_fuel(departure: (f64, f64), arrival: (f64, f64), consumption_per_km: f64) -> f64 {
    let distance_km = haversine(departure.0, departure.1, arrival.0, arrival.1) / 1000.0;
    distance_km * consumption_per_km
}
```

**6. Maritime Navigation**
```rust
// Estimate time to destination at constant speed
fn eta_hours(ship_pos: (f64, f64), dest: (f64, f64), speed_knots: f64) -> f64 {
    let distance_nm = haversine(ship_pos.0, ship_pos.1, dest.0, dest.1) / 1852.0; // nautical miles
    distance_nm / speed_knots
}
```

**7. Geographic Clustering**
```rust
// K-means clustering with geographic distance
fn geo_kmeans(points: &[(f64, f64)], k: usize) -> Vec<Vec<(f64, f64)>> {
    // Use haversine as distance metric instead of Euclidean
    // Implementation would use haversine for centroid calculation
    unimplemented!()
}
```

### 6.2 Related Algorithms

**When to Use Haversine**:
- ✅ General-purpose distance calculation
- ✅ Short to medium distances (< 1000 km)
- ✅ Real-time applications requiring speed
- ✅ Acceptable 0.5% error margin
- ✅ Global coverage

**Alternatives**:

| Algorithm | Use Case | Accuracy | Speed |
|-----------|----------|----------|-------|
| **Vincenty Formula** | High-precision geodesy | ±0.5 mm | Slower |
| **Karney's Method** | Near-antipodal points | ±15 nm | Medium |
| **Equirectangular** | Very short distances | ±1% | Fastest |
| **Law of Cosines** | Educational purposes | Similar | Similar |
| **Great Circle (spherical law of cosines)** | Legacy code | Poor for small d | Similar |

**Vincenty Formula** (ellipsoidal model):
```rust
// More accurate but computationally expensive
// Accounts for Earth's ellipsoidal shape (WGS84)
// Error: ±0.5 mm vs ±0.5% for Haversine
// Use for: Surveying, high-precision navigation
```

**Equirectangular Approximation** (flat Earth):
```rust
// Fast approximation for small distances
// Error increases with distance and latitude
// Use for: Real-time filtering, UI updates
fn equirectangular_approx(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let x = (lng2 - lng1) * ((lat1 + lat2) / 2.0 * PI / 180.0).cos();
    let y = lat2 - lat1;
    (x * x + y * y).sqrt() * 111.32 * 1000.0 // degrees to meters
}
```

**Bearing + Distance Calculation**:
```rust
// If you need both bearing and distance, compute together
// Saves redundant trigonometric calculations
fn bearing_and_distance(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> (f64, f64) {
    // Reuse intermediate values for efficiency
    (bearing, distance)
}
```

## 7. Testing Strategies

### 7.1 Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_zero_distance() {
        assert_eq!(haversine(52.0, 4.0, 52.0, 4.0), 0.0);
    }
    
    #[test]
    fn test_known_distance() {
        // Amsterdam Centraal to nearby location
        let dist = haversine(52.375603, 4.903206, 52.366059, 4.926692);
        assert!((dist - 1919.0).abs() < 1.0); // ±1 meter tolerance
    }
    
    #[test]
    fn test_equatorial_distance() {
        // 1 degree longitude at equator ≈ 111.32 km
        let dist = haversine(0.0, 0.0, 0.0, 1.0);
        assert!((dist - 111320.0).abs() < 100.0);
    }
    
    #[test]
    fn test_meridian_distance() {
        // 1 degree latitude ≈ 110.57 km
        let dist = haversine(0.0, 0.0, 1.0, 0.0);
        assert!((dist - 110570.0).abs() < 100.0);
    }
    
    #[test]
    fn test_antipodal_points() {
        // Maximum distance (half Earth's circumference)
        let dist = haversine(0.0, 0.0, 0.0, 180.0);
        assert!((dist - 20015086.0).abs() < 1000.0); // π × R
    }
}
```

### 7.2 Property-Based Tests

```rust
#[cfg(test)]
mod property_tests {
    use quickcheck::{quickcheck, TestResult};
    
    quickcheck! {
        // Distance is non-negative
        fn prop_non_negative(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> TestResult {
            if !is_valid_coord(lat1, lng1) || !is_valid_coord(lat2, lng2) {
                return TestResult::discard();
            }
            TestResult::from_bool(haversine(lat1, lng1, lat2, lng2) >= 0.0)
        }
        
        // Distance is symmetric
        fn prop_symmetric(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> TestResult {
            if !is_valid_coord(lat1, lng1) || !is_valid_coord(lat2, lng2) {
                return TestResult::discard();
            }
            let d1 = haversine(lat1, lng1, lat2, lng2);
            let d2 = haversine(lat2, lng2, lat1, lng1);
            TestResult::from_bool((d1 - d2).abs() < 1e-6)
        }
        
        // Triangle inequality
        fn prop_triangle_inequality(
            lat1: f64, lng1: f64,
            lat2: f64, lng2: f64,
            lat3: f64, lng3: f64
        ) -> TestResult {
            if !is_valid_coord(lat1, lng1) || !is_valid_coord(lat2, lng2) || !is_valid_coord(lat3, lng3) {
                return TestResult::discard();
            }
            let d12 = haversine(lat1, lng1, lat2, lng2);
            let d23 = haversine(lat2, lng2, lat3, lng3);
            let d13 = haversine(lat1, lng1, lat3, lng3);
            TestResult::from_bool(d13 <= d12 + d23 + 1.0) // +1.0 for floating-point tolerance
        }
    }
}
```

## 8. Common Pitfalls

### 8.1 Degree vs Radian Confusion
```rust
// ❌ Wrong: Using degrees directly
let a = ((lat2 - lat1) / 2.0).sin().powf(2.0); // WRONG!

// ✅ Correct: Convert to radians first
let delta_rad = (lat2 - lat1) * PI / 180.0;
let a = (delta_rad / 2.0).sin().powf(2.0);
```

### 8.2 Not Validating Coordinates
```rust
// ❌ Wrong: No validation
pub fn haversine(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 { ... }

// ✅ Better: Validate inputs
pub fn haversine_checked(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> Result<f64, &'static str> {
    if !(-90.0..=90.0).contains(&lat1) || !(-90.0..=90.0).contains(&lat2) {
        return Err("Latitude must be between -90 and 90 degrees");
    }
    if !(-180.0..=180.0).contains(&lng1) || !(-180.0..=180.0).contains(&lng2) {
        return Err("Longitude must be between -180 and 180 degrees");
    }
    Ok(haversine(lat1, lng1, lat2, lng2))
}
```

### 8.3 Ignoring Longitude Wrap-Around
```rust
// ❌ Problem: 179°E to -179°W should be 2°, not 358°
haversine(0.0, 179.0, 0.0, -179.0) // Calculates wrong path

// ✅ Solution: Normalize longitude difference
fn normalize_longitude_diff(diff: f64) -> f64 {
    let mut d = diff;
    while d > 180.0 { d -= 360.0; }
    while d < -180.0 { d += 360.0; }
    d
}
```

### 8.4 Using Wrong Earth Radius
```rust
// Different contexts use different radii:
const EARTH_RADIUS_MEAN: f64 = 6371000.0;      // General use
const EARTH_RADIUS_EQUATORIAL: f64 = 6378137.0; // WGS84 equator
const EARTH_RADIUS_POLAR: f64 = 6356752.0;      // WGS84 poles

// Be consistent with your application's requirements
```

### 8.5 Precision Loss for Very Short Distances
```rust
// ❌ Problem: Haversine can lose precision for d < 1 meter
// ✅ Solution: Use law of cosines for very short distances
fn distance_short(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let d = haversine(lat1, lng1, lat2, lng2);
    if d < 10.0 { // Use equirectangular for < 10m
        return equirectangular_approx(lat1, lng1, lat2, lng2);
    }
    d
}
```

## 9. References

### 9.1 Academic Papers
1. **Sinnott, R. W.** (1984). "Virtues of the Haversine". *Sky and Telescope*, 68(2), 159.
   - Original popularization of the haversine formula for navigation

2. **Vincenty, T.** (1975). "Direct and Inverse Solutions of Geodesics on the Ellipsoid with Application of Nested Equations". *Survey Review*, 23(176), 88-93.
   - More accurate ellipsoidal alternative

3. **Karney, C. F. F.** (2013). "Algorithms for geodesics". *Journal of Geodesy*, 87(1), 43-55.
   - Modern improvements for edge cases

### 9.2 Books
1. **Meeus, J.** (1998). *Astronomical Algorithms* (2nd ed.). Willmann-Bell.
   - Chapter on spherical astronomy includes haversine derivation

2. **Snyder, J. P.** (1987). *Map Projections: A Working Manual*. USGS Professional Paper 1395.
   - Comprehensive coverage of geodetic calculations

3. **Iliffe, J. & Lott, R.** (2008). *Datums and Map Projections for Remote Sensing, GIS and Surveying* (2nd ed.). Whittles Publishing.

### 9.3 Online Resources
1. **Movable Type Scripts**: [www.movable-type.co.uk/scripts/latlong.html](https://www.movable-type.co.uk/scripts/latlong.html)
   - Excellent interactive explanations and implementations

2. **Aviation Formulary V1.47** by Ed Williams: [www.edwilliams.org/avform147.htm](http://www.edwilliams.org/avform147.htm)
   - Comprehensive aviation navigation formulas

3. **GeographicLib**: [geographiclib.sourceforge.io](https://geographiclib.sourceforge.io/)
   - High-precision geodesic calculations (Karney's algorithms)

4. **NIST Digital Library of Mathematical Functions**: [dlmf.nist.gov](https://dlmf.nist.gov/)
   - Authoritative mathematical reference

### 9.4 Related Implementations
- **geopy** (Python): Geographic calculation library
- **geolib** (JavaScript): Client-side geo calculations
- **geographiclib** (C++): High-precision geodesic library
- **proj** (C): Cartographic projections library

### 9.5 Standards
- **WGS84**: World Geodetic System 1984 (GPS standard)
- **EPSG:4326**: WGS84 coordinate reference system
- **ISO 6709**: Standard representation of geographic coordinates
