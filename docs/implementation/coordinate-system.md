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

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [Dependency graph](./dependency-graph.md) | [Coordinate system reference](../reference/coordinate-system.md)
