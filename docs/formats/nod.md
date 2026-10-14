# NOD — Nihilistic Object Data

**Extension:** `.nod`
**Type:** Binary
**Version:** `flags` field ≥ 10

3D model format with skeleton, multiple vertex types, indexed triangle meshes, and material references.

## File Layout

```
[Header: 92 bytes (0x5C)]
  uint32  flags          Version/feature flags (≥ 10)
  uint8   shaderCount    Number of material names
  uint8   boneCount      Skeleton bone count
  uint8   vertexSetCount Number of vertex buffer sets
  uint8   meshGroupCount Number of draw groups
  uint32  unknown
  vec3    boundsMin      AABB minimum
  vec3    boundsMax      AABB maximum
  ...     (padding/reserved to 92 bytes)
  uint32  indexCount     Total index count

[Shader Names: shaderCount × 32 bytes]
  Null-padded material name strings

[Bones: boneCount × 64 bytes]
  float[3]  position     Rest-pose translation
  float[9]  orientation  3×3 rotation matrix (row-major)
  float[3]  offset       Inverse root translation
  uint8     pad0         Secondary link / IK parent
  uint8     pad1         Joint mapping tag
  uint8     parent       Parent bone index (0xFF = root)
  uint8     jointId      Joint identifier

[Vertex Data: variable]
  Per vertex set: count × stride bytes
  Stride determined by vertex type

[Index Buffer: indexCount × uint16]
  Triangle strip and list indices

[Mesh Groups: meshGroupCount × 56 bytes]
  uint32  shaderIndex    Material index
  Strip/list index ranges
  uint32  baseVertex     Base vertex offset
  uint8   vertexSetIndex Which vertex set to use
  uint8   numBones       Bone palette size
  uint8[20] aBones       Bone palette indices
```

## Vertex Types

| Type | Total bytes | Layout | Description |
|------|-------------|--------|-------------|
| 0 | 32 | pos(12) + normal(12) + uv(8) | Rigid |
| 1 | 36 | pos(12) + normal(12) + uv(8) + boneIdx(4) | Single-bone skinned |
| 2 | 48 | pos(12) + normal(12) + uv(8) + weights(12) + boneIdx(4) | Weighted skinned |
| 3 | 68 | [stream 0: 32B = type 0] + [stream 1: 36B TBN] | Bump-mapped rigid |
| 4 | 72 | [stream 0: 36B = type 1] + [stream 1: 36B TBN] | Bump-mapped single-bone |
| 5 | 84 | [stream 0: 48B = type 2] + [stream 1: 36B TBN] | Bump-mapped weighted |

**Sequential stream layout (types 3–5):** Types 3–5 store vertex data as **two sequential (non-interleaved) streams**:
- **Stream 0:** `count × baseStride` bytes — identical layout to the base type (3→0, 4→1, 5→2)
- **Stream 1:** `count × 36` bytes — TBN data: tangent(12) + binormal(12) + normal(12), all unit vectors

The total byte count per vertex set is `count × totalStride`, but stream 0 vertex data must be read at the base stride (32/36/48), not the combined stride.

**Type prevalence:** Type 0 (1379 files), Type 1 (154), Type 2 (102), Type 3 (11), Type 4 (3), Type 5 (1 — lurker.nod).

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NAD](./nad.md) | [NSA](./nsa.md) | [NOC](./noc.md) | [NMB](./nmb.md) | [Model Format Relationships](../implementation/model-relationships.md) | [NOD Vertex Data Layouts](../implementation/nod-vertices.md) | [NOD Index Buffer Encoding](../implementation/nod-indices.md) | [Coordinate system](../reference/coordinate-system.md)
