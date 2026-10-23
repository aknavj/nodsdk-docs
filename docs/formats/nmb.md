# NMB — Nihilistic Model Bundle

**Extension:** `.nmb`
**Type:** Binary
**Version:** 1 or 2

Multi-model bundle format containing multiple LOD variants of models. 40 files in `3D/Models/`, each paired with an `.nnb` animation bundle.

## File Layout

```
[Header: 12 bytes]
  uint32  version      1 or 2
  uint32  modelCount   Number of models in bundle
  uint32  totalMem     Native 32-bit CPU object-arena allocation size

[Per Entry]
  [Compact Header: 5 bytes]
    uint8   shaderCount
    uint8   boneCount
    uint8   meshGroupCount
    uint16  markerCount

  [Entry Data: variable]
    char[32]  name           Model name
    uint8     vertexSetCount
    uint32    flags
    float[3]  boundsMin
    float[3]  boundsMax

    [Vertex Sets]
      Per set: uint8 type + uint32 count + count×stride vertex data

    [Index Buffer]
      uint32 indexCount + indexCount × uint16

    [Shader Names: shaderCount × 32 bytes]

    [Bones: boneCount × 26 bytes]
      float[3] position (12B) + float[3] offset (12B) + uint8 parent + uint8 jointId

    [Blend Data]
      uint32 blendCount + blendCount × 24 bytes

    [Mesh Groups: meshGroupCount entries, variable]
      Additional per-group: uint8 originalGroup, numBlendShapes, groupLinkIndex, jointMapSize
      + jointMapSize bytes bone remap table

    [Markers: markerCount × 68 bytes]
      byte[4] packedIdentity
      float[16] matrix

    [Version 2 Color Stream]
      uint8 hasColors       0 or 1; not generic padding
      If hasColors == 1:
        byte[][4] colors    One packed color per vertex across all vertex sets
```

## Vertex Types (same as NOD)

| Type | Stride | Layout |
|------|--------|--------|
| 0 | 32B | pos + normal + uv |
| 1 | 36B | pos + normal + uv + boneIdx |
| 2 | 48B | pos + normal + uv + weights + boneIdx |
| 3 | 68B | stream 0 (type 0, 32B) + stream 1 (TBN, 36B) |
| 4 | 72B | stream 0 (type 1, 36B) + stream 1 (TBN, 36B) |
| 5 | 84B | stream 0 (type 2, 48B) + stream 1 (TBN, 36B) |

Types 3–5 use two sequential (non-interleaved) streams. See NOD vertex types for details.

## Native Object Arena

`totalMem` is neither serialized file length nor GPU buffer size. For the verified
version-2 native layout, sum this allocation for each member:

```
328 + 4*shaderCount + 32*boneCount + 108*meshGroupCount + 80*markerCount
```

Vertex, index, TBN, optional-color, and blend buffers are allocated separately.
These sizes describe the native 32-bit runtime, independent of the host
tool's pointer width. The formula matches 39 of 40 supplied bundle headers.
`Escape_Mini.nmb` reports 480 bytes less than this allocation requires; new
output calculates the native requirement rather than reproducing that anomaly.

## Verified v10 NOD/NOC to v2 NMB Profile

The verified conversion profile uses flags `0x00000A00`. NOC members retain
their `.noc` identities and collision geometry; they must not be substituted
with same-stem visible NOD geometry.

The source 56-byte mesh-group record maps to the compact bundle record as follows
(source byte ranges are half-open):

| Source bytes | Bundle meaning |
|--------------|----------------|
| 0..4 | Shader index |
| 4..28 | Four LOD records |
| 28..30 | Source vertex advance; must match LOD-0 vertex count in this profile |
| 30..33 | Original-group, blend-shape count, and group-link bytes |
| 33..53 | Bone-palette bytes, truncated to the declared palette size |
| 53 | Palette size |
| 54 | Vertex-set index |

Per-set vertex bases are reconstructed from group advances, making a mismatch
with LOD-0 counts unsafe. The current writer rejects unsupported blend-shape
and reserved extensions; that is a bounded writer restriction, not a universal
native-format prohibition.

Source markers' four float3 rows become float4 rows by appending `0, 0, 0, 1`,
retaining their four-byte identity. Bundle bones retain position, offset, parent,
and joint-map bytes in their 26-byte representation; source orientation matrices
and padding are not additional bundled bone fields. Optional source vertex
colors are retained through the version-2 color stream.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOD](./nod.md) | [NNB](./nnb.md) | [Model Format Relationships](../implementation/model-relationships.md)
