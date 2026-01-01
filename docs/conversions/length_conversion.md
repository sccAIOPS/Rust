# Length Unit Conversion

## 1. Overview

Length unit conversion transforms measurements between different units of length, supporting both metric (millimeter, centimeter, meter, kilometer) and imperial (inch, foot, yard, mile) systems. This is a fundamental operation in scientific computing, engineering, international commerce, and everyday applications.

**Historical Context**: Length measurement systems evolved independently across civilizations. The metric system was established in 1795 during the French Revolution, based on decimal multiples. The imperial system derives from British units standardized in 1824. The 1959 International Yard and Pound Agreement unified imperial and US customary units. Today, most countries use the metric system, though the US, Liberia, and Myanmar primarily use imperial units.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a length value $L$ in unit $U_{\text{from}}$, convert it to equivalent length $L'$ in unit $U_{\text{to}}$.

**Formal Definition**:

$$L' = L \times \frac{M_{\text{from}}}{M_{\text{to}}}$$

where $M_u$ is the conversion factor from unit $u$ to a base unit (meter).

**Two-Step Conversion**:
1. Convert from source unit to meters: $L_{\text{meter}} = L \times M_{\text{from}}$
2. Convert from meters to target unit: $L' = L_{\text{meter}} / M_{\text{to}}$

### 2.2 Mathematical Model

**Supported Units**:

| Unit | Symbol | Meter Conversion | System |
|------|--------|------------------|--------|
| Millimeter | mm | 0.001 m | Metric |
| Centimeter | cm | 0.01 m | Metric |
| Meter | m | 1.0 m | Metric (base) |
| Kilometer | km | 1000 m | Metric |
| Inch | in | 0.0254 m | Imperial |
| Foot | ft | 0.3048 m | Imperial |
| Yard | yd | 0.9144 m | Imperial |
| Mile | mi | 1609.34 m | Imperial |

**Input/Output Specifications**:
- Input: `(value: f64, from_unit: LengthUnit, to_unit: LengthUnit)`
- Output: `f64` (converted value)
- All conversions preserve precision using `f64`

**Key Properties**:
1. **Transitivity**: If $A \to B$ and $B \to C$, then $A \to C$
2. **Reflexivity**: Converting $A \to A$ yields identity
3. **Proportionality**: Conversion is linear (no offset like temperature)
4. **Precision**: Uses exact conversion factors from international standards

### 2.3 Correctness Proof

**Lemma 1 (Identity)**: For any unit $u$ and value $v$:
$$\text{convert}(v, u, u) = v$$

Proof: 
$$v \times \frac{M_u}{M_u} = v \times 1 = v$$

**Lemma 2 (Transitivity)**: For units $u_1, u_2, u_3$ and value $v$:
$$\text{convert}(\text{convert}(v, u_1, u_2), u_2, u_3) = \text{convert}(v, u_1, u_3)$$

Proof:
$$v \times \frac{M_1}{M_2} \times \frac{M_2}{M_3} = v \times \frac{M_1}{M_3}$$

**Theorem (Correctness)**: The two-step conversion through meters is equivalent to direct conversion.

Proof:
$$\frac{L \times M_{\text{from}}}{M_{\text{to}}} = L \times \frac{M_{\text{from}}}{M_{\text{to}}}$$

## 3. Algorithm Description

### 3.1 Intuition

The algorithm uses a "hub-and-spoke" pattern with meters as the central hub. Instead of maintaining $n \times n$ conversion factors for $n$ units, we only need $n$ factors (each unit to meters). Any conversion follows the path:

```
Source Unit → Meters → Target Unit
```

This simplifies the implementation and ensures consistency.

**Example**: Converting 12 inches to centimeters
1. Inches to meters: $12 \times 0.0254 = 0.3048$ meters
2. Meters to centimeters: $0.3048 / 0.01 = 30.48$ centimeters

### 3.2 Pseudocode

```
ENUM LengthUnit:
    Millimeter, Centimeter, Meter, Kilometer
    Inch, Foot, Yard, Mile
END ENUM

FUNCTION unit_to_meter_multiplier(unit):
    MATCH unit:
        Millimeter → 0.001
        Centimeter → 0.01
        Meter → 1.0
        Kilometer → 1000.0
        Inch → 0.0254
        Foot → 0.3048
        Yard → 0.9144
        Mile → 1609.34
    END MATCH
END FUNCTION

FUNCTION unit_to_meter(value, from_unit):
    RETURN value × unit_to_meter_multiplier(from_unit)
END FUNCTION

FUNCTION meter_to_unit(value, to_unit):
    RETURN value / unit_to_meter_multiplier(to_unit)
END FUNCTION

FUNCTION length_conversion(value, from_unit, to_unit):
    // Step 1: Convert to meters
    meters ← unit_to_meter(value, from_unit)
    
    // Step 2: Convert from meters to target
    result ← meter_to_unit(meters, to_unit)
    
    RETURN result
END FUNCTION
```

**Line-by-line annotations**:
- `unit_to_meter_multiplier`: Lookup table for conversion factors
- `unit_to_meter`: Multiply by factor to convert to base unit
- `meter_to_unit`: Divide by factor to convert from base unit
- `length_conversion`: Two-step conversion process

### 3.3 Step-by-Step Example

**Example 1**: Convert 5 feet to meters

| Step | Operation | Value |
|------|-----------|-------|
| Input | 5 feet | 5.0 |
| 1. Lookup factor | Foot → Meter | 0.3048 |
| 2. Multiply | 5 × 0.3048 | 1.524 |
| 3. Already in meters | - | 1.524 |

**Result**: 1.524 meters

**Example 2**: Convert 100 inches to centimeters

| Step | Operation | Value |
|------|-----------|-------|
| Input | 100 inches | 100.0 |
| 1. Inch → Meter factor | 0.0254 | - |
| 2. Convert to meters | 100 × 0.0254 | 2.54 |
| 3. Meter → Cm factor | 0.01 | - |
| 4. Convert to cm | 2.54 / 0.01 | 254.0 |

**Result**: 254.0 centimeters

**Example 3**: Convert 1 mile to kilometers

| Step | Operation | Calculation |
|------|-----------|-------------|
| 1. Mile → Meter | 1 × 1609.34 | 1609.34 m |
| 2. Meter → Km | 1609.34 / 1000 | 1.60934 km |

**Result**: 1.60934 kilometers

## 4. Complexity Analysis

### 4.1 Time Complexity

- **All Cases**: $O(1)$ - Constant time
- Fixed number of operations:
  1. Pattern match (constant time in Rust)
  2. One multiplication
  3. One division
- Independent of input value magnitude

### 4.2 Space Complexity

- **Auxiliary Space**: $O(1)$
- Only stores:
  - Conversion factors (compile-time constants)
  - Intermediate meters value
  - No dynamic allocation

**Total Space**: $O(1)$ - constant regardless of input

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Enum Definition**:
```rust
#[derive(Clone, Copy, PartialEq, Eq, Hash)]
pub enum LengthUnit {
    Millimeter, Centimeter, Meter, Kilometer,
    Inch, Foot, Yard, Mile,
}
```

**Traits**:
- `Clone, Copy`: Efficient copying (enum is small)
- `PartialEq, Eq`: Equality comparison
- `Hash`: Can be used as HashMap keys

**Pattern Matching**:
```rust
fn unit_to_meter_multiplier(from: LengthUnit) -> f64 {
    match from {
        LengthUnit::Millimeter => 0.001,
        LengthUnit::Centimeter => 0.01,
        // ... exhaustive matching ensures safety
    }
}
```

**Function Composition**:
```rust
pub fn length_conversion(input: f64, from: LengthUnit, to: LengthUnit) -> f64 {
    meter_to_unit(unit_to_meter(input, from), to)
}
```

**Precision Considerations**:
- Uses `f64` for maximum precision (53-bit mantissa)
- Conversion factors are exact representations where possible
- Some imperial→metric conversions have rounding errors

**Zero Handling**:
- Zero in any unit → Zero in any other unit
- Preserved by identity: $0 \times k / k = 0$
- Test verifies: `length_conversion(0.0, u1, u2) == 0.0` for all units

### 5.2 Edge Cases

1. **Zero Values**:
   - 0 in any unit → 0 in any other unit
   - Test: `zero_to_zero()` verifies all combinations

2. **Identity Conversions**:
   - Same source and target unit
   - Should return exact input value
   - May have floating-point rounding

3. **Precision Loss**:
   - Very small values (e.g., 0.001 mm → mile)
   - Very large values (e.g., 10^15 miles → mm)
   - Floating-point underflow/overflow

4. **Round-Trip Accuracy**:
   ```rust
   let original = 100.0;
   let converted = length_conversion(original, Meter, Foot);
   let back = length_conversion(converted, Foot, Meter);
   assert!((original - back).abs() < 0.0000001);
   ```

5. **Exact Equivalences**:
   - 1 meter = 100 centimeters (exact)
   - 1 foot = 12 inches (exact)
   - 1 mile = 5280 feet (exact)
   - 1 inch = 2.54 cm (exact by definition since 1959)

6. **Asymmetric Precision**:
   - Metric-to-metric: Often exact
   - Imperial-to-imperial: Often exact
   - Cross-system: May have small errors

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Geographic Information Systems (GIS)**:
   - Map projection calculations
   - Distance measurements on maps
   - GPS coordinate calculations
   - Example: Converting GPS distances to display units

2. **Engineering Design Software**:
   - CAD/CAM systems (AutoCAD, SolidWorks)
   - Structural engineering calculations
   - Manufacturing specifications
   - Cross-country team collaboration

3. **E-Commerce Platforms**:
   - International shipping dimensions
   - Product size displays
   - Warehouse management systems
   - Example: Converting package dimensions for different markets

4. **Scientific Computing**:
   - Physics simulations (particle distances)
   - Astronomy calculations (astronomical units)
   - Chemistry (molecular dimensions)
   - Experiment data processing

5. **Navigation Systems**:
   - GPS navigation (route distances)
   - Aviation (nautical miles, feet)
   - Maritime navigation
   - Running/cycling apps

6. **Construction and Architecture**:
   - Building plans (blueprints)
   - Material calculations
   - Code compliance (different standards)
   - International projects

7. **Health and Fitness**:
   - Running distance tracking
   - Height/weight measurements
   - Medical equipment readings
   - Sports statistics

8. **Weather and Climate**:
   - Rainfall measurements
   - Snow depth reporting
   - Visibility distance
   - International weather data exchange

### 6.2 Related Algorithms

**Other Unit Conversions**:

| Type | Units | Complexity | Notes |
|------|-------|------------|-------|
| Length | mm, cm, m, km, in, ft, yd, mi | O(1) | Linear conversion |
| Mass | g, kg, lb, oz | O(1) | Linear conversion |
| Volume | mL, L, cup, gal | O(1) | Linear conversion |
| Temperature | °C, °F, K | O(1) | **Non-linear** (offset) |
| Speed | m/s, km/h, mph | O(1) | Compound units |
| Area | m², ft², acre | O(1) | Squared conversion |

**Temperature is Different**:
```
Celsius ↔ Fahrenheit: F = C × 9/5 + 32
Length: No offset term!
```

**Compound Unit Conversions**:
- Speed: Length/Time (e.g., m/s → mph)
- Density: Mass/Volume (e.g., kg/m³ → lb/ft³)
- Requires conversion of numerator AND denominator

**Design Patterns**:

1. **Hub-and-Spoke** (This implementation):
   - All conversions through base unit (meter)
   - $n$ conversion factors for $n$ units
   - Easy to add new units

2. **Direct Conversion Table**:
   - $n \times n$ lookup table
   - $O(1)$ lookup, but $O(n^2)$ space
   - Harder to maintain consistency

3. **Graph-Based**:
   - Units as nodes, conversions as edges
   - Pathfinding for multi-step conversions
   - Overkill for simple linear units

**When to Use Each**:
- **Hub-and-spoke**: Length, mass, volume (proportional)
- **Offset conversions**: Temperature, pressure (affine)
- **Graph-based**: Currency (time-varying exchange rates)
- **Lookup tables**: Small, fixed set (shoe sizes)

**Extended Functionality**:

```rust
// Arithmetic with units (dimensional analysis)
struct Length {
    value: f64,
    unit: LengthUnit,
}

impl Length {
    fn add(&self, other: &Length) -> Length {
        let other_converted = length_conversion(
            other.value, other.unit, self.unit
        );
        Length {
            value: self.value + other_converted,
            unit: self.unit,
        }
    }
}
```

## 7. References

1. **Bureau International des Poids et Mesures (BIPM)**. *The International System of Units (SI)* (9th ed., 2019). [Official SI Brochure](https://www.bipm.org/en/publications/si-brochure/)

2. **NIST Special Publication 811**: *Guide for the Use of the International System of Units (SI)*. National Institute of Standards and Technology, 2008.

3. **Wikipedia**: [Conversion of Units](https://en.wikipedia.org/wiki/Conversion_of_units)

4. **Wikipedia**: [Metric System](https://en.wikipedia.org/wiki/Metric_system)

5. **Wikipedia**: [Imperial Units](https://en.wikipedia.org/wiki/Imperial_units)

6. **International Yard and Pound Agreement (1959)**: Standardized conversion: 1 yard = 0.9144 meters exactly

7. **ISO 80000-3:2019**: Quantities and units - Part 3: Space and time

8. **NIST Reference on Constants, Units and Uncertainty**: [Unit Conversions](https://physics.nist.gov/cuu/Reference/unitconversions.html)

9. **Rust num_traits crate**: For numeric trait abstractions in generic unit conversion systems

10. **Dimensioned crate**: Rust library for compile-time dimensional analysis

11. **uom (units of measurement) crate**: Comprehensive Rust library for type-safe unit conversions
