# NIL — Nihilistic Indoor Level

**Extension:** `.nil`
**Type:** Binary
**Magic:** `0x104C494E` ("NIL\x10")
**Version:** 35

The primary level geometry format. Contains sector-based BSP/portal architecture with per-vertex baked lighting, lightmap patches, and environment map references.

## File Layout

```
[Header]
  uint32  magic            "NIL\x10"
  uint32  version          35
  uint8[72] globalLight    Global light parameters
  float[3]  ambientColor   RGB ambient color
  uint32  numShaders       Material count
  char[]  shaderNames      numShaders × 33-byte padded names
  uint16  numEnvMaps       Environment map count
  char[]  envMapNames      numEnvMaps × 33-byte padded names
  uint32  numSectors       Sector count

[Sectors: numSectors entries]
  Per sector (variable size):
    uint32    flags
    float[3]  center       Sector center point
    uint8     numPortals   Portal count
    uint8[N]  portalData   Portal definitions (N × 16 bytes)
    uint8     hasGeo       0=empty, 1=standard, 2=with lightmaps
    uint8     vertType     0=standard, 1=with TBN, 2=vertex color
    uint16    numVerts     Vertex count

    [Vertex Data: numVerts × 36 bytes]
      float[3]  position   World-space position
      float[3]  normal     Surface normal
      uint32    diffuse    BGRA packed vertex color (Xbox D3DCOLOR)
      float[2]  uv         Texture coordinates

    [TBN Data: numVerts × 36 bytes, if vertType == 1]
      float[3]  tangent
      float[3]  binormal
      float[3]  normal

    [Secondary Vertex Pool: if hasGeo >= 2]
      uint8   secondaryVertType  Observed: 1 (with TBN)
      uint32  secondaryVertCount
      byte[]  vertices           secondaryVertCount × 36 bytes
      byte[]  tbn                secondaryVertCount × 36 bytes
      Previously labelled "lightmap patches" by the C++ parser.
      NILtoMAP now selects this pool for bumped groups (vertType 3)
      when the primary vertex pool has no TBN stream. Verified against
      Hive level indices and group bounds; other secondary types remain
      unsupported by the tool.

    uint16    numGroups    Surface group count
    [Surface Groups: numGroups entries, 47 bytes each]
      uint32  shaderIdx        Material index
      uint32  startIdx         Index buffer start
      uint32  field8
      uint16  numStripIndices  Triangle strip index count
      uint16  numListIndices   Triangle list index count
      uint32  field16
      uint8   vertType         Vertex type override
      uint8   flag             Rendering flags
      uint8   field22
      float[3] bboxMin         Group AABB minimum
      float[3] bboxMax         Group AABB maximum

    uint32    numIndices   Index count
    uint16[]  indices      Triangle strip/list indices

    [Things: variable]
      uint8   numThings
      [Thing: 68 bytes each]
        float[3]  position
        uint32    field1
        uint32    field2
        float[12] matrix    4×vec3 row matrix

    [SurfThings: variable]
      uint16  numSurfThings
      [SurfThing: 72 bytes each]
        uint8[72] data      Opaque

    [EnvRefs: variable]
      uint16  numEnvRefs
      char[]  envRefNames   N × 33-byte names

    [Tail: 6 bytes]
      uint16  tailField1   Unknown
      uint32  tailField2   Unknown

[Post-Sector Data: variable size, ~11–42% of file]
  The data after the sector loop contains spatial/lighting structures
  that have NOT been fully decoded. Summary from hex analysis:

  [Section 1: Portal Collision Mesh]
    uint32       marker      Always 0
    float[6]     aabb        Level bounding box (min xyz, max xyz), LE
    uint8        type        Always 1
    uint8        pad         Always 0
    uint8        flag        Always 1
    uint8        vertCount   Number of collision vertices
    float[3][]   vertices    vertCount × 12 bytes, LE float3 positions
                             Forms axis-aligned box faces (portal boundaries)
    uint16       tail        Unknown (e.g. 0x90C0)

  [Section 2: Triangle Indices]
    uint8[~5]    zeros       Section separator
    uint16[]     indices     Triangle indices into Section 1 vertices
                             Pattern: (2,1,0), (3,2,4), (6,5,4), ...

  [Section 3+: BSP Tree / Lightmap Atlas — NOT DECODED]
    ~99% of the post-sector data. Byte entropy 7.0–7.3 / 8.0
    indicates compressed texture data, likely the lightmap atlas.
    Contains:
    - Scattered float3 positions (world coordinates)
    - AABB-like patterns throughout (~200+ in first 100KB)
    - Possible BSP tree node data (splitting planes + child refs)
    - Dense high-entropy regions (lightmap pixel data)

  [Tail: Offset Table]
    Last ~500–600 bytes of the file.
    Pairs of (uint32 offset, uint32 flag) where flag ∈ {0, 1}.
    Values are monotonically increasing within groups.
    Groups separated by flag=0 entries.
    Likely sector-indexed offsets into the lightmap/BSP data.
```

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

Related: [NSA](./nsa.md) | [NSD](./nsd.md) | [NPD](./npd.md) | [NLFX](./nlfx.md) | [Level Format Relationships](../implementation/level-relationships.md) | [NIL Vertex Data Layout](../implementation/nil-vertices.md) | [Coordinate system](../reference/coordinate-system.md)
