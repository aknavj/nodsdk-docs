# NOD Vertex Data Layouts

NOD stores vertices in three formats depending on skinning requirements:

## Type 0 — Rigid (32 bytes/vertex)

| Offset | Size | Field |
|---|---|---|
| 0 | 12 | position (float[3]) |
| 12 | 12 | normal (float[3]) |
| 24 | 8 | uv (float[2]) |

No bone assignment — vertex is static or transformed as a whole object.

## Type 1 — Single Bone (36 bytes/vertex)

| Offset | Size | Field |
|---|---|---|
| 0 | 12 | position (float[3]) |
| 12 | 12 | normal (float[3]) |
| 24 | 8 | uv (float[2]) |
| 32 | 4 | boneIndex (uint32, local palette index) |

Vertex is rigidly attached to one bone. Weight is implicitly 1.0.

## Type 2 — Weighted (48 bytes/vertex)

| Offset | Size | Field |
|---|---|---|
| 0 | 12 | position (float[3]) |
| 12 | 12 | normal (float[3]) |
| 24 | 8 | uv (float[2]) |
| 32 | 12 | weights (float[3], fourth weight = 1.0 − sum) |
| 44 | 4 | boneIndices (uint8[4], local palette indices) |

Up to 4 bones influence each vertex. Indices reference the mesh group's local bone palette (`aBones[20]`), which maps to global skeleton indices.

## Bone Palette Indirection

```
  vertex.boneIndex[i]         local palette index (0–19)
        │
        ▼
  meshGroup.aBones[local]     global skeleton bone index (0–boneCount)
        │
        ▼
  NOD bone array[global]      rest pose position + orientation + parent
```

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NOD](../formats/nod.md) | [Dependency graph](./dependency-graph.md)
