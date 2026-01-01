# Bearing Calculation

## 1. Overview

The **bearing** (also called **azimuth**) is the direction or angle from one geographic point to another, measured clockwise from true north. This algorithm calculates the **initial bearing** (forward azimuth) at the starting point when traveling along the great circle route between two points on Earth's surface.

Historically essential for navigation before GPS, bearing calculations remain crucial for:
- Ship and aircraft navigation
- Hiking and orienteering
- Mapping and surveying
- Autonomous vehicle navigation
- Directional indicators in mobile apps

Unlike the simple trigonometric bearing on a flat surface, geographic bearing must account for Earth's spherical geometry, where meridians (lines of longitude) converge at the poles, causing the bearing to change continuously along a great circle route.

**Important Note**: This algorithm computes the **initial bearing** at the departure point. On a great circle path, the bearing changes continuously. For navigation along a constant bearing (rhumb line), see the [rhumbline.md](rhumbline.md) documentation.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: Two points on Earth's surface
- Point 1 (start): $(lat_1, lng_1)$ in degrees
- Point 2 (end): $(lat_2, lng_2)$ in degrees

**Output**: Initial bearing $\theta$ in degrees (0° to 360°)
- 0° = North
- 90° = East
- 180° = South
- 270° = West

**Convention**: Bearing is measured clockwise from true north (not magnetic north)

### 2.2 Mathematical Model

The bearing formula derives from spherical trigonometry on a unit sphere:

$$\theta = \text{atan2}\left(\sin(\Delta\lambda) \cdot \cos(\phi_2), \cos(\phi_1) \cdot \sin(\phi_2) - \sin(\phi_1) \cdot \cos(\phi_2) \cdot \cos(\Delta\lambda)\right)$$

After computing in radians, convert to degrees and normalize:

$$\theta_{\text{deg}} = (\theta_{\text{rad}} \times \frac{180}{\pi} + 360) \mod 360$$

Where:
- $\phi_1, \phi_2$ = latitude of point 1 and 2 (in radians)
- $\lambda_1, \lambda_2$ = longitude of point 1 and 2 (in radians)
- $\Delta\lambda = \lambda_2 - \lambda_1$ (longitude difference)
- $\text{atan2}(y, x)$ = four-quadrant inverse tangent

### 2.3 Derivation from Spherical Trigonometry

Consider a spherical triangle formed by:
- The North Pole
- Point 1 (start)
- Point 2 (end)

Using the **spherical law of cosines for sides** and **law of sines**, we derive:

$$\tan(\theta) = \frac{\sin(\Delta\lambda)}{\cos(\phi_1) \cdot \tan(\phi_2) - \sin(\phi_1) \cdot \cos(\Delta\lambda)}$$

This is reformulated using $\text{atan2}$ to handle all quadrants correctly without division:

$$y = \sin(\Delta\lambda) \cdot \cos(\phi_2)$$
$$x = \cos(\phi_1) \cdot \sin(\phi_2) - \sin(\phi_1) \cdot \cos(\phi_2) \cdot \cos(\Delta\lambda)$$
$$\theta = \text{atan2}(y, x)$$

### 2.4 Why atan2 Instead of atan?

The standard $\arctan$ function has a range of $[-\frac{\pi}{2}, \frac{\pi}{2}]$ (only two quadrants), losing directional information.

The $\text{atan2}(y, x)$ function:
- Returns values in $[-\pi, \pi]$ (all four quadrants)
- Handles signs of both $x$ and $y$ to determine correct quadrant
- Avoids division by zero when $x = 0$
- Preserves directional information

Example:
```
atan(1/1) = 45°        // Could be NE or SW
atan2(1, 1) = 45°      // Definitely NE
atan2(-1, -1) = -135°  // Definitely SW
```

### 2.5 Mathematical Properties

**Property 1: Non-symmetric**
$$\text{bearing}(A \to B) \neq \text{bearing}(B \to A)$$
Unlike distance, bearing depends on direction.

**Property 2: Discontinuous at North Pole**
Bearing is undefined at the poles (all directions are south/north).

**Property 3: Changes Along Great Circle**
The bearing changes continuously along a great circle path (except for meridian lines).

**Property 4: Constant Only on Rhumb Lines**
Only rhumb lines (loxodromes) maintain constant bearing.

## 3. Algorithm Description

### 3.1 Intuition

Imagine you're standing at Point A with a compass, facing Point B:

1. **North Reference**: First, establish true north (0°)
2. **Spherical Adjustment**: Account for Earth's curvature using spherical trigonometry
3. **Longitude Difference**: Calculate how far east/west the destination is
4. **Latitude Difference**: Account for how far north/south the destination is
5. **Quadrant Resolution**: Use atan2 to determine the correct compass direction
6. **Normalization**: Convert result to standard 0-360° range

The algorithm essentially projects the destination point onto a plane tangent to Earth at your location, then calculates the angle, adjusting for spherical distortion.

### 3.2 Pseudocode

```
FUNCTION Bearing(lat1, lng1, lat2, lng2):
    // Constants
    PI = 3.14159...
    
    // Step 1: Convert degrees to radians
    φ1 = lat1 × π / 180
    φ2 = lat2 × π / 180
    λ1 = lng1 × π / 180
    λ2 = lng2 × π / 180
    
    // Step 2: Calculate longitude difference
    Δλ = λ2 - λ1
    
    // Step 3: Calculate bearing components (spherical trig)
    y = sin(Δλ) × cos(φ2)
    x = cos(φ1) × sin(φ2) - sin(φ1) × cos(φ2) × cos(Δλ)
    
    // Step 4: Calculate bearing in radians
    θ_rad = atan2(y, x)
    
    // Step 5: Convert to degrees
    θ_deg = θ_rad × 180 / π
    
    // Step 6: Normalize to 0-360°
    bearing = (θ_deg + 360) mod 360
    
    RETURN bearing
END FUNCTION
```

### 3.3 Step-by-Step Example

**Problem**: Calculate the bearing from a point in southern Brazil to a point in northern Brazil.

**Input**:
- Point A (start): (-27.2020447088982°S, -49.631891179172555°W)
- Point B (end): (-3.106362°S, -60.025826°W)

**Step 1**: Convert to radians
```
φ1 = -27.2020447088982 × π/180 = -0.474760 rad
φ2 = -3.106362 × π/180 = -0.054220 rad
λ1 = -49.631891179172555 × π/180 = -0.866253 rad
λ2 = -60.025826 × π/180 = -1.047445 rad
```

**Step 2**: Calculate longitude difference
```
Δλ = λ2 - λ1 = -1.047445 - (-0.866253) = -0.181192 rad
```

**Step 3**: Calculate bearing components
```
sin(Δλ) = sin(-0.181192) = -0.180185
cos(φ2) = cos(-0.054220) = 0.998529

y = -0.180185 × 0.998529 = -0.179920

cos(φ1) = cos(-0.474760) = 0.889577
sin(φ2) = sin(-0.054220) = -0.054188
sin(φ1) = sin(-0.474760) = -0.456789
cos(Δλ) = cos(-0.181192) = 0.983605

x = 0.889577 × (-0.054188) - (-0.456789) × 0.998529 × 0.983605
x = -0.048210 + 0.448458
x = 0.400248
```

**Step 4**: Calculate bearing in radians
```
θ_rad = atan2(-0.179920, 0.400248) = -0.420735 rad
```

**Step 5**: Convert to degrees
```
θ_deg = -0.420735 × 180/π = -24.109°
```

**Step 6**: Normalize to 0-360°
```
bearing = (-24.109 + 360) mod 360 = 335.891° ≈ 336°
```

**Interpretation**: The bearing is approximately **336°**, which is **NNW** (north-northwest). This makes sense: traveling from southern Brazil (Point A at 27°S) to northern Brazil (Point B at 3°S) while also going west (from -49°W to -60°W) results in a northwesterly direction.

## 4. Complexity Analysis

### 4.1 Time Complexity

**O(1)** - Constant time

The algorithm performs a fixed number of operations regardless of input:
- 4 multiplications for degree-to-radian conversion
- 1 subtraction for longitude difference
- 6 trigonometric function calls (sin, cos)
- 1 atan2 function call
- 3 arithmetic operations (multiply, subtract, multiply)
- 1 modulo operation for normalization

All operations are primitive and execute in constant time.

### 4.2 Space Complexity

**O(1)** - Constant space

The algorithm uses only a fixed number of variables:
- 4 input coordinates
- 4 converted radian values
- 1 longitude difference
- 2 bearing components (x, y)
- 2 result values (radians, degrees)

No data structures or dynamic memory allocation.

### 4.3 Performance Characteristics

- **Execution Time**: ~60-120 nanoseconds on modern CPUs
- **Cache Efficiency**: All data fits in L1 cache
- **Branch Prediction**: No conditional branches in main path
- **Vectorization**: Can compute multiple bearings simultaneously (SIMD)
- **Numerical Stability**: atan2 is numerically stable for all inputs

### 4.4 Computational Cost Breakdown

| Operation | Count | Relative Cost | Total |
|-----------|-------|---------------|-------|
| Multiplication | 7 | 1 | 7 |
| Division | 4 | 3 | 12 |
| Subtraction | 2 | 1 | 2 |
| sin/cos | 6 | 50 | 300 |
| atan2 | 1 | 80 | 80 |
| Modulo | 1 | 3 | 3 |
| **Total** | | | **~400 cycles** |

Trigonometric functions dominate the cost (~95% of execution time).

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::f64::consts::PI;

pub fn bearing(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    // Convert to radians
    let lat1 = lat1 * PI / 180.0;
    let lng1 = lng1 * PI / 180.0;
    let lat2 = lat2 * PI / 180.0;
    let lng2 = lng2 * PI / 180.0;
    
    // Calculate longitude difference
    let delta_longitude = lng2 - lng1;
    
    // Calculate bearing components
    let y = delta_longitude.sin() * lat2.cos();
    let x = lat1.cos() * lat2.sin() - lat1.sin() * lat2.cos() * delta_longitude.cos();
    
    // Calculate bearing
    let mut brng = y.atan2(x);
    brng = brng.to_degrees();
    
    // Normalize to 0-360°
    (brng + 360.0) % 360.0
}
```

**Key Features**:
- **Variable Shadowing**: Reuses variable names (`lat1`, `lng1`, etc.) for converted radians
  - Prevents accidental use of degree values after conversion
  - Reduces namespace pollution
  - Common Rust idiom for unit conversions
  
- **Method Chaining**: 
  ```rust
  delta_longitude.sin()  // Method on f64
  brng.to_degrees()      // Built-in conversion
  ```
  
- **atan2 Method**: Rust's `f64::atan2(y, x)` matches mathematical convention
  - Note: Some languages reverse the argument order
  
- **Modulo with Floats**: `%` operator works with `f64` in Rust
  - Alternative: `brng.rem_euclid(360.0)` for always-positive remainder

**Type Information**:
- Input: Four `f64` parameters
- Output: `f64` bearing in degrees (0.0 to 360.0)
- No generic types needed (specific to geographic coordinates)
- No lifetime parameters (all inputs are copy types)

### 5.2 Edge Cases

**1. Same Point (Undefined Bearing)**
```rust
bearing(52.0, 4.0, 52.0, 4.0)
// Returns: Some angle (mathematically undefined)
// x = 0, y = 0, atan2(0, 0) = 0 in Rust
// Better: Check if points are equal first
```

**2. North-South Along Meridian**
```rust
bearing(0.0, 0.0, 10.0, 0.0)  // Due north
// Returns: 0° (correct)

bearing(10.0, 0.0, 0.0, 0.0)  // Due south
// Returns: 180° (correct)
```

**3. East-West Along Equator**
```rust
bearing(0.0, 0.0, 0.0, 10.0)  // Due east
// Returns: 90° (correct)

bearing(0.0, 10.0, 0.0, 0.0)  // Due west
// Returns: 270° (correct)
```

**4. Across Date Line (Antimeridian)**
```rust
bearing(0.0, 179.0, 0.0, -179.0)
// Calculates correctly: atan2 handles wrapped angles
// Returns: ~90° (east, taking shorter path)
```

**5. Near Poles**
```rust
bearing(89.9, 0.0, 89.9, 180.0)
// At high latitudes, bearing changes rapidly
// Numerically stable but may surprise users
```

**6. Exactly at North Pole**
```rust
bearing(90.0, 0.0, 85.0, 0.0)
// From North Pole, all directions are south
// Returns: 180° regardless of longitude
```

**7. Exactly at South Pole**
```rust
bearing(-90.0, 0.0, -85.0, 0.0)
// From South Pole, all directions are north
// Returns: 0° or 360° (equivalent)
```

**8. Invalid Coordinates (No Validation)**
```rust
bearing(200.0, 500.0, 300.0, 600.0)
// No bounds checking; computes nonsense result
// Production code should validate inputs
```

### 5.3 Production-Ready Wrapper

```rust
use std::f64::consts::PI;

#[derive(Debug, Clone, Copy, PartialEq)]
pub struct GeoCoord {
    pub lat: f64,
    pub lng: f64,
}

impl GeoCoord {
    /// Create new coordinate with validation
    pub fn new(lat: f64, lng: f64) -> Result<Self, &'static str> {
        if !(-90.0..=90.0).contains(&lat) {
            return Err("Latitude must be between -90 and 90 degrees");
        }
        if !(-180.0..=180.0).contains(&lng) {
            return Err("Longitude must be between -180 and 180 degrees");
        }
        Ok(Self { lat, lng })
    }
    
    /// Calculate initial bearing to another point
    pub fn bearing_to(&self, other: &GeoCoord) -> Option<f64> {
        // Check if points are effectively the same (within 1 meter)
        const EPSILON: f64 = 0.00001; // ~1 meter
        if (self.lat - other.lat).abs() < EPSILON && 
           (self.lng - other.lng).abs() < EPSILON {
            return None; // Undefined bearing
        }
        
        Some(bearing(self.lat, self.lng, other.lat, other.lng))
    }
    
    /// Get compass direction string
    pub fn compass_direction(&self, other: &GeoCoord) -> Option<&'static str> {
        self.bearing_to(other).map(|brng| {
            match brng {
                b if b < 22.5 || b >= 337.5 => "N",
                b if b < 67.5 => "NE",
                b if b < 112.5 => "E",
                b if b < 157.5 => "SE",
                b if b < 202.5 => "S",
                b if b < 247.5 => "SW",
                b if b < 292.5 => "W",
                _ => "NW",
            }
        })
    }
}

// Core algorithm (unchanged)
fn bearing(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let lat1 = lat1 * PI / 180.0;
    let lng1 = lng1 * PI / 180.0;
    let lat2 = lat2 * PI / 180.0;
    let lng2 = lng2 * PI / 180.0;
    
    let delta_longitude = lng2 - lng1;
    let y = delta_longitude.sin() * lat2.cos();
    let x = lat1.cos() * lat2.sin() - lat1.sin() * lat2.cos() * delta_longitude.cos();
    
    let mut brng = y.atan2(x);
    brng = brng.to_degrees();
    (brng + 360.0) % 360.0
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_bearing_wrapper() {
        let a = GeoCoord::new(-27.202, -49.632).unwrap();
        let b = GeoCoord::new(-3.106, -60.026).unwrap();
        
        assert_eq!(a.bearing_to(&b).unwrap().round(), 336.0);
        assert_eq!(a.compass_direction(&b), Some("NNW"));
    }
    
    #[test]
    fn test_same_point_returns_none() {
        let a = GeoCoord::new(52.0, 4.0).unwrap();
        assert_eq!(a.bearing_to(&a), None);
    }
    
    #[test]
    fn test_invalid_coordinates() {
        assert!(GeoCoord::new(91.0, 0.0).is_err());
        assert!(GeoCoord::new(0.0, 181.0).is_err());
    }
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Turn-by-Turn Navigation**
```rust
/// Generate turn instructions based on bearing change
fn generate_instruction(from: GeoCoord, via: GeoCoord, to: GeoCoord) -> String {
    let bearing1 = from.bearing_to(&via).unwrap_or(0.0);
    let bearing2 = via.bearing_to(&to).unwrap_or(0.0);
    let turn_angle = (bearing2 - bearing1 + 360.0) % 360.0;
    
    match turn_angle {
        a if a < 15.0 || a > 345.0 => "Continue straight".to_string(),
        a if a < 45.0 => "Bear slight right".to_string(),
        a if (45.0..135.0).contains(&a) => "Turn right".to_string(),
        a if (135.0..225.0).contains(&a) => "Make U-turn".to_string(),
        a if (225.0..315.0).contains(&a) => "Turn left".to_string(),
        _ => "Bear slight left".to_string(),
    }
}
```

**2. Compass Widget in Mobile Apps**
```rust
/// Display direction to destination
struct CompassView {
    user_location: GeoCoord,
    destination: GeoCoord,
    device_heading: f64, // From device compass
}

impl CompassView {
    fn arrow_rotation(&self) -> f64 {
        let bearing = self.user_location.bearing_to(&self.destination).unwrap_or(0.0);
        // Rotate arrow relative to device heading
        (bearing - self.device_heading + 360.0) % 360.0
    }
    
    fn display_text(&self) -> String {
        let direction = self.user_location.compass_direction(&self.destination).unwrap_or("--");
        format!("{} →", direction)
    }
}
```

**3. Autopilot Systems**
```rust
/// Calculate required heading for waypoint navigation
fn calculate_course_correction(
    current_pos: GeoCoord,
    current_heading: f64,
    next_waypoint: GeoCoord
) -> f64 {
    let desired_heading = current_pos.bearing_to(&next_waypoint).unwrap_or(current_heading);
    let mut correction = desired_heading - current_heading;
    
    // Normalize to [-180, 180] for shortest turn
    if correction > 180.0 {
        correction -= 360.0;
    } else if correction < -180.0 {
        correction += 360.0;
    }
    
    correction // Positive = turn right, Negative = turn left
}
```

**4. Drone Flight Path Planning**
```rust
/// Generate waypoints with smooth direction changes
fn optimize_flight_path(waypoints: &[GeoCoord]) -> Vec<(GeoCoord, f64)> {
    waypoints.windows(2)
        .map(|w| {
            let bearing = w[0].bearing_to(&w[1]).unwrap_or(0.0);
            (w[1], bearing)
        })
        .collect()
}
```

**5. Augmented Reality POI Indicators**
```rust
/// Determine if POI is visible in camera view
struct ARCamera {
    position: GeoCoord,
    heading: f64,      // Device compass heading
    field_of_view: f64, // Camera FOV in degrees
}

impl ARCamera {
    fn is_poi_visible(&self, poi: &GeoCoord) -> bool {
        let bearing = self.position.bearing_to(poi).unwrap_or(0.0);
        let relative_bearing = (bearing - self.heading + 360.0) % 360.0;
        
        // Check if within FOV (centered at 0°)
        relative_bearing <= self.field_of_view / 2.0 || 
        relative_bearing >= 360.0 - self.field_of_view / 2.0
    }
}
```

**6. Maritime Collision Avoidance**
```rust
/// Check if two vessels are on collision course
fn collision_risk(
    ship1: (GeoCoord, f64), // Position and heading
    ship2: (GeoCoord, f64),
    time_horizon_seconds: f64,
    speed_knots: f64
) -> bool {
    let bearing_1_to_2 = ship1.0.bearing_to(&ship2.0).unwrap_or(0.0);
    let bearing_2_to_1 = ship2.0.bearing_to(&ship1.0).unwrap_or(0.0);
    
    // If each ship's heading points toward the other (opposite bearings)
    let heading_diff_1 = (ship1.1 - bearing_1_to_2).abs();
    let heading_diff_2 = (ship2.1 - bearing_2_to_1).abs();
    
    heading_diff_1 < 15.0 && heading_diff_2 < 15.0 // Within 15° tolerance
}
```

**7. Solar Panel Orientation**
```rust
/// Calculate optimal solar panel azimuth toward the sun
fn solar_panel_bearing(panel_location: GeoCoord, sun_position: GeoCoord) -> f64 {
    // sun_position calculated from time and date
    panel_location.bearing_to(&sun_position).unwrap_or(180.0) // Default south
}
```

**8. Photography Planning (Golden Hour)**
```rust
/// Find bearing to sunset for landscape photography
fn sunset_bearing(location: GeoCoord, date: Date) -> f64 {
    let sun_position = calculate_sun_position(location, date, "sunset");
    location.bearing_to(&sun_position).unwrap_or(270.0) // Default west
}
```

### 6.2 Related Algorithms

**When to Use Bearing Calculation**:
- ✅ Initial direction for route planning
- ✅ Compass UI indicators
- ✅ Turn-by-turn navigation instructions
- ✅ Autopilot heading calculations
- ✅ AR/VR directional indicators
- ❌ Long-distance navigation (bearing changes along great circle)
- ❌ Constant-direction paths (use rhumb line bearing)

**Comparison with Rhumb Line Bearing**:

| Feature | Great Circle Bearing | Rhumb Line Bearing |
|---------|---------------------|-------------------|
| **Path Type** | Shortest distance | Constant bearing |
| **Bearing Stability** | Changes continuously | Remains constant |
| **Use Case** | Initial direction, aviation | Maritime navigation |
| **Calculation** | Faster (simpler) | Slightly more complex |
| **Distance** | Shorter | Longer (except meridians/equator) |
| **Practicality** | Requires continuous adjustment | Easier to navigate manually |

**Related Functions to Implement Together**:

```rust
/// Complete navigation toolkit
pub trait NavigationTools {
    /// Initial bearing (this algorithm)
    fn bearing_to(&self, other: &GeoCoord) -> Option<f64>;
    
    /// Great circle distance (Haversine)
    fn distance_to(&self, other: &GeoCoord) -> f64;
    
    /// Rhumb line bearing (constant)
    fn rhumb_bearing_to(&self, other: &GeoCoord) -> f64;
    
    /// Rhumb line distance
    fn rhumb_distance_to(&self, other: &GeoCoord) -> f64;
    
    /// Destination point given bearing and distance
    fn destination(&self, bearing: f64, distance: f64) -> GeoCoord;
    
    /// Midpoint between two coordinates
    fn midpoint_to(&self, other: &GeoCoord) -> GeoCoord;
    
    /// Final bearing (bearing when arriving at destination)
    fn final_bearing_to(&self, other: &GeoCoord) -> Option<f64> {
        // Reverse bearing from destination to origin, then add 180°
        other.bearing_to(self).map(|b| (b + 180.0) % 360.0)
    }
}
```

**Bearing vs. Heading vs. Track**:
- **Bearing**: Direction *to* a target from current position
- **Heading**: Direction the vehicle is *pointing*
- **Track**: Direction of actual *movement* (may differ due to wind/current)

## 7. Testing Strategies

### 7.1 Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_bearing_north() {
        let brng = bearing(0.0, 0.0, 1.0, 0.0);
        assert!((brng - 0.0).abs() < 0.1); // Due north
    }
    
    #[test]
    fn test_bearing_east() {
        let brng = bearing(0.0, 0.0, 0.0, 1.0);
        assert!((brng - 90.0).abs() < 0.1); // Due east
    }
    
    #[test]
    fn test_bearing_south() {
        let brng = bearing(1.0, 0.0, 0.0, 0.0);
        assert!((brng - 180.0).abs() < 0.1); // Due south
    }
    
    #[test]
    fn test_bearing_west() {
        let brng = bearing(0.0, 1.0, 0.0, 0.0);
        assert!((brng - 270.0).abs() < 0.1); // Due west
    }
    
    #[test]
    fn test_bearing_northeast() {
        let brng = bearing(0.0, 0.0, 1.0, 1.0);
        assert!(brng > 0.0 && brng < 90.0); // NE quadrant
    }
    
    #[test]
    fn test_bearing_range() {
        let brng = bearing(-27.202, -49.632, -3.106, -60.026);
        assert!(brng >= 0.0 && brng < 360.0); // Valid range
        assert!((brng - 336.0).abs() < 1.0); // Expected value
    }
    
    #[test]
    fn test_bearing_across_dateline() {
        // From 179°E to -179°W (should go east, not west around)
        let brng = bearing(0.0, 179.0, 0.0, -179.0);
        assert!((brng - 90.0).abs() < 5.0); // Roughly east
    }
    
    #[test]
    fn test_bearing_high_latitude() {
        // Near North Pole
        let brng = bearing(85.0, 0.0, 85.0, 90.0);
        assert!(brng >= 0.0 && brng < 360.0);
    }
}
```

### 7.2 Property-Based Tests

```rust
#[cfg(test)]
mod property_tests {
    use quickcheck::{quickcheck, TestResult};
    use super::*;
    
    fn is_valid_coord(lat: f64, lng: f64) -> bool {
        lat.is_finite() && lng.is_finite() &&
        lat >= -90.0 && lat <= 90.0 &&
        lng >= -180.0 && lng <= 180.0
    }
    
    quickcheck! {
        // Bearing is always in [0, 360)
        fn prop_bearing_range(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> TestResult {
            if !is_valid_coord(lat1, lng1) || !is_valid_coord(lat2, lng2) {
                return TestResult::discard();
            }
            let brng = bearing(lat1, lng1, lat2, lng2);
            TestResult::from_bool(brng >= 0.0 && brng < 360.0)
        }
        
        // Bearing + 180° ≈ reverse bearing (for short distances)
        fn prop_reverse_bearing(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> TestResult {
            if !is_valid_coord(lat1, lng1) || !is_valid_coord(lat2, lng2) {
                return TestResult::discard();
            }
            // Skip if points are very close (bearing undefined)
            if (lat1 - lat2).abs() < 0.01 && (lng1 - lng2).abs() < 0.01 {
                return TestResult::discard();
            }
            
            let forward = bearing(lat1, lng1, lat2, lng2);
            let backward = bearing(lat2, lng2, lat1, lng1);
            let diff = ((forward + 180.0) % 360.0 - backward).abs();
            
            // Allow some tolerance for great circle effects
            TestResult::from_bool(diff < 5.0 || diff > 355.0)
        }
        
        // Cardinal directions from origin
        fn prop_cardinal_directions(distance: f64) -> TestResult {
            if distance <= 0.0 || distance > 10.0 {
                return TestResult::discard();
            }
            
            let north = bearing(0.0, 0.0, distance, 0.0);
            let east = bearing(0.0, 0.0, 0.0, distance);
            let south = bearing(distance, 0.0, 0.0, 0.0);
            let west = bearing(0.0, distance, 0.0, 0.0);
            
            TestResult::from_bool(
                north < 1.0 &&
                (east - 90.0).abs() < 1.0 &&
                (south - 180.0).abs() < 1.0 &&
                (west - 270.0).abs() < 1.0
            )
        }
    }
}
```

### 7.3 Integration Tests

```rust
#[cfg(test)]
mod integration_tests {
    use super::*;
    
    #[test]
    fn test_navigation_workflow() {
        // Simulate complete navigation workflow
        let start = GeoCoord::new(51.5074, -0.1278).unwrap(); // London
        let end = GeoCoord::new(48.8566, 2.3522).unwrap();    // Paris
        
        // Step 1: Calculate initial bearing
        let initial_bearing = start.bearing_to(&end).unwrap();
        assert!((initial_bearing - 140.0).abs() < 5.0); // Southeast
        
        // Step 2: Check compass direction
        let direction = start.compass_direction(&end);
        assert_eq!(direction, Some("SE"));
        
        // Step 3: Calculate distance (integration with haversine)
        let distance = haversine(start.lat, start.lng, end.lat, end.lng);
        assert!(distance > 340000.0 && distance < 350000.0); // ~344 km
    }
    
    #[test]
    fn test_circumnavigation() {
        // Test bearings around the world
        let coords = vec![
            (0.0, 0.0),    // Equator/Prime Meridian
            (0.0, 90.0),   // Equator/90°E
            (0.0, 180.0),  // Equator/Date Line
            (0.0, -90.0),  // Equator/90°W
        ];
        
        for i in 0..coords.len() {
            let next = (i + 1) % coords.len();
            let brng = bearing(coords[i].0, coords[i].1, coords[next].0, coords[next].1);
            assert!((brng - 90.0).abs() < 1.0); // All should be ~90° (east)
        }
    }
}
```

## 8. Common Pitfalls

### 8.1 Confusing Initial and Final Bearing
```rust
// ❌ Wrong: Assuming bearing is constant
let start = GeoCoord::new(0.0, 0.0);
let end = GeoCoord::new(60.0, 0.0);
let bearing = start.bearing_to(&end).unwrap(); // 0° (north)

// But final bearing when arriving is NOT 0°!
let final_bearing = end.bearing_to(&start).map(|b| (b + 180.0) % 360.0).unwrap();
// final_bearing ≈ 180° but not exactly (great circle effect)

// ✅ Correct: Recalculate bearing at each waypoint for accurate navigation
```

### 8.2 Not Normalizing to [0, 360°)
```rust
// ❌ Wrong: Forgetting normalization
pub fn bearing_wrong(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let brng_rad = /* calculation */;
    brng_rad.to_degrees() // Returns [-180, 180], not [0, 360)
}

// ✅ Correct: Always normalize
pub fn bearing_correct(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    let mut brng = /* calculation */.to_degrees();
    (brng + 360.0) % 360.0 // Ensures [0, 360)
}
```

### 8.3 Reversing atan2 Arguments
```rust
// ❌ Wrong: atan2(x, y) instead of atan2(y, x)
let bearing = x.atan2(y); // WRONG!

// ✅ Correct: atan2(y, x) - standard mathematical convention
let bearing = y.atan2(x);

// Note: Some languages/libraries vary! Always check documentation.
```

### 8.4 Using for Long-Distance Constant-Bearing Navigation
```rust
// ❌ Wrong: Using initial bearing for entire journey
let bearing = start.bearing_to(&end).unwrap();
// Then sailing at this constant bearing will NOT reach the destination!

// ✅ Correct: Use rhumb line bearing for constant-bearing navigation
let rhumb_bearing = rhumb_bearing(start.lat, start.lng, end.lat, end.lng);
// This bearing stays constant along the path
```

### 8.5 Not Handling Same-Point Case
```rust
// ❌ Wrong: No check for identical points
pub fn bearing(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> f64 {
    // ... calculates atan2(0, 0) when points are same
}

// ✅ Better: Return Option or check distance
pub fn bearing_safe(lat1: f64, lng1: f64, lat2: f64, lng2: f64) -> Option<f64> {
    const MIN_DISTANCE: f64 = 1.0; // 1 meter
    let dist = haversine(lat1, lng1, lat2, lng2);
    if dist < MIN_DISTANCE {
        return None; // Bearing undefined
    }
    Some(bearing(lat1, lng1, lat2, lng2))
}
```

### 8.6 Degree/Radian Confusion
```rust
// ❌ Wrong: Mixing degrees and radians
let bearing_rad = calculate_bearing(); // Returns radians
display_ui(bearing_rad); // Displays to user without conversion!

// ✅ Correct: Use type system to prevent confusion
#[derive(Debug, Clone, Copy)]
struct Degrees(f64);
#[derive(Debug, Clone, Copy)]
struct Radians(f64);

impl From<Radians> for Degrees {
    fn from(rad: Radians) -> Self {
        Degrees(rad.0 * 180.0 / PI)
    }
}
```

### 8.7 Ignoring Magnetic Declination
```rust
// ❌ Wrong: Using true bearing with magnetic compass
let true_bearing = calculate_bearing(); // True north
follow_magnetic_compass(true_bearing); // Compass uses magnetic north!

// ✅ Correct: Apply magnetic declination
fn true_to_magnetic(true_bearing: f64, declination: f64) -> f64 {
    (true_bearing - declination + 360.0) % 360.0
}

// Declination varies by location and time (update from model)
```

## 9. References

### 9.1 Academic Papers
1. **Bowditch, N.** (2017). *The American Practical Navigator*. National Geospatial-Intelligence Agency.
   - Chapter 11: Navigational Mathematics
   - Comprehensive coverage of bearing calculations

2. **Williams, E. D.** (n.d.). "Aviation Formulary V1.47".
   - Section on great circle navigation
   - Bearing and distance formulas

3. **Tobler, W.** (1961). "Map Transformations of Geographic Space". *University of Washington*.
   - Foundations of coordinate transformations

### 9.2 Books
1. **Smart, W. M.** (1977). *Textbook on Spherical Astronomy* (6th ed.). Cambridge University Press.
   - Mathematical foundations of spherical trigonometry

2. **Van Sickle, J.** (2008). *Basic GIS Coordinates* (2nd ed.). CRC Press.
   - Practical applications of geographic calculations

3. **Hofmann-Wellenhof, B., et al.** (2007). *Navigation: Principles of Positioning and Guidance*. Springer.
   - Modern navigation systems and algorithms

### 9.3 Online Resources
1. **Movable Type Scripts**: [www.movable-type.co.uk/scripts/latlong.html](https://www.movable-type.co.uk/scripts/latlong.html)
   - Interactive bearing calculator with visualizations
   - JavaScript reference implementation

2. **NOAA - Magnetic Declination**: [www.ngdc.noaa.gov/geomag/calculators](https://www.ngdc.noaa.gov/geomag/calculators)
   - Magnetic declination calculator for true-to-magnetic conversion

3. **Great Circle Mapper**: [www.gcmap.com](http://www.gcmap.com/)
   - Visualize great circle paths and bearings

4. **FAA Navigation Handbook**: [www.faa.gov](https://www.faa.gov/)
   - Aviation navigation procedures using bearing

### 9.4 Standards and Specifications
- **ISO 6709**: Standard representation of geographic point location
- **WGS84**: World Geodetic System (geographic coordinate standard)
- **ICAO Annex 15**: Aeronautical Information Services (bearing conventions)
- **IMO SOLAS**: Maritime navigation requirements (bearing usage)

### 9.5 Related Libraries
- **geographiclib** (C++): High-precision geodesic calculations
- **pyproj** (Python): Cartographic projections and transformations
- **turf.js** (JavaScript): Geospatial analysis library
- **geo-rust** (Rust): Geographic algorithms ecosystem
