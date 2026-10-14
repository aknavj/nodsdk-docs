# NOC — Nihilistic Object Collision

**Extension:** `.noc`
**Type:** Binary
**Version:** 10

Per-model collision mesh. Contains simplified geometry used for physics/collision detection.

## File Layout

```
[Header: 88 bytes (0x58)]
  uint32  version      Always 10
  uint8[4] flags       flags[0]=name mode, flags[3]=shape count
  vec3    boundsMin    AABB minimum
  vec3    boundsMax    AABB maximum
  ...     (additional header fields)

[Shape Descriptor: 100 or 132 bytes]
  Depends on flags[0]:
    flags[0]=1: 100 bytes (name only)
      4B pad + 32B name + 12B pad + 36B matrix + 12B position + 4B color
    flags[0]=2: 132 bytes (name + collision type)
      4B pad + 32B shapeName + 32B collisionType + 12B pad + 36B matrix + 12B position + 4B color

[Vertex Data: numVertices × 32 bytes]
  float[3]  position   Vertex position
  float[3]  normal     Vertex normal
  float[2]  uv         Texture coordinates

[Index Data: numIndices × uint16]
  Triangle list indices

[Footer: 64 + (shapeCount-1) × 56 bytes]
  Additional shape descriptors
```

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOD](./nod.md) | [Model Format Relationships](../implementation/model-relationships.md) | [Coordinate system](../reference/coordinate-system.md)
