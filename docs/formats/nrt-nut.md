# NRT/NUT — Nihilistic Terrain

**Extension:** `.nrt`, `.nut`
**Type:** Binary

Terrain heightmap and texture data organized as a grid of patches.

## File Layout

```
[Header: 8 bytes]
  uint32  version
  uint16  gridWidth
  uint16  gridHeight

[Patches: variable]
  Per patch:
    char[256]  name           Patch/texture name
    float[4]   scaleMatrix    2×2 transform matrix
    uint32     dataSize       Raw patch data size
    uint8[]    data           Heightmap/blend data
```

The `.nrt` file contains the geometry/heightmap data while `.nut` contains the texture layer assignments. Both share the same grid dimensions.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NSA](./nsa.md) | [Coordinate system](../reference/coordinate-system.md)
