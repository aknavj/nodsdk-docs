# Coordinate System

All spatial data uses the **Direct3D left-handed Z-up** coordinate system:

| Axis | Direction |
|------|-----------|
| X | Right |
| Y | Forward (into screen) |
| Z | Up |

NPD navigation nodes use these same **world-space XYZ** positions without
additional XY scaling or Z compression. For model-local data, use the
[static transform contract](../implementation/templates.md#static-model-transforms)
before comparing it with world-space placements or navigation nodes.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [Spatial data conventions](../implementation/coordinate-system.md)
