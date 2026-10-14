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
  uint32  totalMem     Total memory estimate

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
    [Padding: 1 byte]
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

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOD](./nod.md) | [NNB](./nnb.md) | [Model Format Relationships](../implementation/model-relationships.md)
