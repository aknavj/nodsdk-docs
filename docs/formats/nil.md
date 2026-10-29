# NIL — Nihilistic Indoor Level

**Extension:** `.nil`
**Type:** Binary
**Magic:** `0x104C494E` ("NIL\x10")
**Version:** 35

Sector-based level geometry, per-vertex baked lighting, static lights for moving
objects, environment references, and Havok collision data. The verified v35
layout does **not** contain a separate lightmap texture atlas. Earlier
interpretations of secondary vertex pools or post-sector bytes as lightmaps
are superseded by the native loader framing.

## File Layout

```
[Header]
  uint32  magic            "NIL\x10"
  uint32  version          35
  uint8[72] globalLight    Global light record; bytes 8..79
  float[3]  ambientColor   RGB ambient floats; bytes 80..91
  uint32  numShaders       Material count
  char[]  shaderNames      numShaders × 32-byte names
  uint16  numEnvMaps       Environment map count
  char[]  envMapNames      numEnvMaps × 32-byte names
  uint32  numSectors       Sector count

[Sectors: numSectors entries]
  Per sector (variable size):
    uint32    flags
    float[3]  center       Sector center point
    uint8     numBoundaryPlanes
    float[4][] boundaryPlanes   numBoundaryPlanes × 16 bytes
    uint8     numVertexPools    Observed: 0, 1 or 2

    [For each vertex pool]
      uint8   poolType       0=primary stream, 1=primary plus TBN stream
      uint32  numVerts       Native reader consumes 4 bytes and uses low 16 bits

      [Vertex Data: numVerts × 36 bytes]
        float[3] position
        float[3] normal
        uint32   diffuse    Packed Xbox D3DCOLOR
        float[2] uv

      [TBN Data: numVerts × 36 bytes, only for poolType == 1]
        float[3] tangent
        float[3] binormal
        float[3] normal

    uint16    numGroups    Surface group count
    [Surface Groups: numGroups entries, 47 bytes each]
      uint32  shaderIdx        Material index
      uint32  startIdx         Index buffer start
      uint32  field8           Unknown; observed 0
      uint16  numStripIndices  Triangle strip index count
      uint16  numListIndices   Triangle list index count
      uint32  field16          Unknown; observed 0
      uint16  renderFlags      Not a vertex-format selector
      uint8   vertexPool       Index into the owning sector's vertex pools
      float[3] bboxMin         Group AABB minimum
      float[3] bboxMax         Group AABB maximum

    uint32    numIndices   Present even when numVertexPools is zero
    uint16[]  indices      Triangle strip/list indices

    [Things: variable]
      uint8   numThings
      [Thing: 68 bytes each]
        float[3]  position
        uint32    field1
        uint32    field2
        float[12] matrix    4×vec3 row matrix

    [Static Lights: variable]
      uint16  numStaticLights
      [tRenderLight: 72 bytes each]
        uint8[72] data      Native light record, formerly named "SurfThing"

    [EnvRefs: variable]
      uint16  numEnvRefs
      char[]  envRefNames   numEnvRefs × 32-byte names

    [Tail: 6 bytes]
      uint16  tailField1   Unknown; observed 0
      uint32  tailField2   Unknown; observed 1

[Post-Sector Data: verified no-terrain profile]
  uint16 terrainCount       0 in the eight examined original files
  uint16 physicsSetup       Observed 0; meaning not inferred
  float[6] physicsBounds    Min xyz, max xyz
  uint16 havokVersion       1

  Repeat:
    uint8 segmentType       0=end, 1=world rigid body
    If segmentType == 1:
      uint16 vertexCount
      float[3][] vertices
      uint32 indexCount
      uint16[] indices      Triangle-list indices
      uint16 materialCount
      byte[][8] materials
      float[4] moppInfo
      uint32 moppCodeSize
      byte[] moppCode
      uint16 ownerType      0=world, 1=sector, 2=terrain
      uint16 ownerIndex     Sector/terrain index when applicable

  For each sector:
    uint32 collisionPresent
    For each render surface group:
      uint32 mappingPresent
      If mappingPresent != 0:
        uint32 collisionOffset
```

Nonzero terrain requires a separate decoder; the zero-terrain layout above
does not describe terrain payloads.

Every sector has a `collisionPresent` word, followed by one `mappingPresent`
word for **every** render surface group, even when `collisionPresent` is zero.
An absent group mapping occupies only its four-byte zero `mappingPresent`
word: there is no `collisionOffset` placeholder. Any nonzero `mappingPresent`
is followed by a four-byte `collisionOffset`. Extra offset words for absent
groups desynchronize later groups/sectors or leave trailing bytes.

## Native Interpretation

- Shader-table entries are native material references. The SDK writes a
  direct DDS basename for direct textures, or the canonical NSA material name
  for an archived definition; it does not replace an archived name with its
  diffuse texture name. Generated names are NUL-terminated ASCII strings of
  1..31 bytes in the 32-byte field. DDS-backed archived materials pass through
  without embedding shader code, material banks or textures.
- Original vertex-pool sequences are `[]`, `[0]`, `[1]`, or `[0, 1]`.
  Observed counts do not exceed 65,535. A group selects a pool by index, not by
  interpreting `renderFlags` as a vertex type. Legacy `hasGeo`, padding, and
  secondary-pool names must not be used to infer a different disk layout.
- Boundary records are inward half-space planes `(nx, ny, nz, d)`. A point is
  in the sector when every plane satisfies `dot(n, p) >= d`. Zero boundary planes
  make that sector cover all space; physics broadphase bounds remain separate.
- `collisionOffset` is an inclusive triangle-key start, not a byte offset or
  an index-buffer offset. Material records and surface mappings have different
  lookup rules; see [NIL collision and spatial data](../implementation/nil-collision.md).
- Static-light records and header ambient affect moving objects separately from
  baked vertex colors; see [NIL lighting](../implementation/nil-lighting.md).

The matching debug executable identifies the loading sequence as
`gameWorld_LoadWorld` (`0x00534410`), `renderTerrain_ReadFile` (`0x005e5670`),
`gameHavok_ReadPreHavokSetup` (`0x00518c40`),
`gameHavok_ReadHavokFile` (`0x00519380`), and
`tWorld::ReadHavokSurfaceGroupOffsets` (`0x00533dc0`).
Mesh and MOPP framing are read by `hkNSIMeshShape::ReadData` (`0x00518d10`) and
`simPhysics_ReadMoppCodeFromFile` (`0x004921b0`). These are build-specific
evidence addresses, not portable API identifiers.

## Per-File Post-Sector Statistics

| File | Sectors | Remaining | % of File |
|------|---------|-----------|-----------|
| 1_2_1_Miners_Bunker.nil | 14 | 973 KB | 30.1% |
| 1_2_1a_Miners_Bunker.nil | 40 | 1,043 KB | 41.9% |
| 1_2_1b_Miners_Bunker.nil | 55 | 1,281 KB | 30.4% |
| 1_3_1a_Hive_StationOuter.nil | 47 | 1,537 KB | 27.6% |
| 1_3_1b_Hive_StationOuter.nil | 302 | 2,046 KB | 29.9% |
| 2_2_2_THREAT_BASEUPPER.nil | 43 | 209 KB | 11.0% |
| 2_2_2a_Threat_BaseUpper.nil | 116 | 509 KB | 12.3% |
| menu.nil | 2 | 790 KB | 34.1% |

## Vertex Color Encoding

Xbox D3DCOLOR is BGRA packed as `uint32`. Vertex colors encode baked lighting with Xbox overbright convention: value 128 = 1.0 intensity, 255 ≈ 2.0 intensity. The renderer scales by `color / 128.0`.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NSA](./nsa.md) | [NSD](./nsd.md) | [NPD](./npd.md) | [NLFX](./nlfx.md) | [Level Format Relationships](../implementation/level-relationships.md) | [NIL Vertex Data Layout](../implementation/nil-vertices.md) | [NIL lighting](../implementation/nil-lighting.md) | [NIL collision](../implementation/nil-collision.md) | [Coordinate system](../reference/coordinate-system.md)
