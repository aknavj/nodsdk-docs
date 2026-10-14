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

## NIL Surface Group Vertex Types

| vertType | Meaning | Extra data |
|---|---|---|
| 0 | Standard (dynamic lighting) | Vertex colors ignored or absent |
| 1 | With TBN basis | Additional 36 bytes: tangent + binormal + normal |
| 2 | Baked vertex color | Vertex colors contain pre-baked lighting |

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [Dependency graph](./dependency-graph.md)
