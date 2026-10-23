# NIL Vertex Data Layout

NIL level vertices encode baked lighting from the Xbox D3DCOLOR format:

## Raw NIL Vertex (36 bytes)

| Offset | Size | Field | Encoding |
|---|---|---|---|
| 0 | 12 | position | float[3], D3D Z-up world space |
| 12 | 12 | normal | float[3], unit vector |
| 24 | 4 | diffuse | uint32 Xbox D3DCOLOR (BGRA packed) |
| 28 | 8 | uv | float[2], texture coordinates |

## Xbox D3DCOLOR Decoding

The `diffuse` field is Xbox D3DCOLOR — BGRA byte order packed into uint32:

```
  Byte layout:  [B][G][R][A]
  Bit layout:   AAAA AAAA RRRR RRRR GGGG GGGG BBBB BBBB

  R = (diffuse >> 16) & 0xFF
  G = (diffuse >>  8) & 0xFF
  B = (diffuse >>  0) & 0xFF
```

Xbox uses **2× overbright** convention: byte value 128 = 1.0 intensity, 255 ≈ 2.0 intensity. To recover linear color: `channel = byte / 128.0`.

## Sector Vertex Pools and Surface Groups

| Pool type | Primary stream | Extra data |
|---|---|---|
| 0 | 36 bytes per vertex, including diffuse | None |
| 1 | Same 36-byte primary vertex | Separate 36-byte-per-vertex tangent/binormal/normal stream |

The sector begins its geometry with a byte-sized pool count. Each pool then
stores a byte-sized type and four-byte vertex count; the native runtime uses
the count's low 16 bits. Original pool sequences are `[]`, `[0]`, `[1]`, and
`[0, 1]`.

Each 47-byte surface group contains **uint16 render flags at byte 20** and a
**uint8 vertex-pool selector at byte 22**. Those fields are not vertex-format
overrides. Select the owning sector's pool by index to interpret the group's
indices and optional TBN data. There is no separate pool type 2 for baked colors:
both primary formats already contain diffuse at byte 24.

`0xffffffff` is an unbaked white baseline. Moving-object lights and header
ambient are separate inputs with different equations; see
[NIL lighting](./nil-lighting.md).

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NIL lighting](./nil-lighting.md) | [NIL collision](./nil-collision.md) | [Dependency graph](./dependency-graph.md)
