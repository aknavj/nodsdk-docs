# Coordinate System

All spatial data across all formats uses **Direct3D left-handed Z-up**:

```
       Z (up)
       │
       │
       └── Y (forward)
      /
     X (right)
```

| Axis | Direction |
|------|-----------|
| X | Right |
| Y | Forward (into screen) |
| Z | Up |

Positions, normals, bone offsets, entity placements, region bounds, and camera keyframes all use this convention.

NPD waypoint positions are also **world-space XYZ**, copied unchanged by the
native loader. They do not have a separate XY scale or compressed Z axis.
See the [NPD layout](../formats/npd.md) for navigation IDs and graph fields.

Static model instances apply
`origin + enabled_offset + rotation * (scale * model_vertex)`.
The enabled model offset is world-translation additive, not rotated or scaled;
see [ordered templates and transforms](./templates.md#static-model-transforms)
for the modifier-order and render-flag requirements.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [Dependency graph](./dependency-graph.md) | [Coordinate system reference](../reference/coordinate-system.md)
