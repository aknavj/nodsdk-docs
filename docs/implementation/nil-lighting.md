# NIL Lighting

World surfaces and moving objects consume separate lighting inputs. Baked
diffuse colors alone do not provide the static lights used by moving NPCs.
See the [NIL format](../formats/nil.md) for their containing records.

## World Vertex Lighting

Each 36-byte primary vertex has packed Xbox D3DCOLOR at byte 24. The native
reader preserves these bytes in the vertex buffer. `0xffffffff` is an unbaked
white baseline, not generated illumination.

The [vertex layout guide](./nil-vertices.md) describes BGRA decoding and the
surface overbright convention. Do not apply that convention as a replacement
for the separate ambient or moving-object light equations below.

## Sector Static Lights

The sector's `uint16` light count is followed by 72-byte `tRenderLight` records:

| Byte | Type | Native use |
|------|------|------------|
| 0 | uint32 | Flags/metadata; complete bit meanings unresolved |
| 4 | uint32 | Light type; sampled original sector point lights use 0 |
| 8 | uint32 | Light-style index; 0 bypasses style lookup |
| 12 | float3 | RGB channels in 0..255 units |
| 24 | float | Render intensity; style-0 addition sets it to input weight times field 28 |
| 28 | float | Selection/base intensity multiplier |
| 32 | float3 | Cone direction |
| 44 | float3 | World position |
| 56 | float2 | Opaque; preserve without inferred meaning |
| 64 | float | Type-3 distance cutoff; not a type-0 object-selection cutoff |
| 68 | float | Falloff coefficient |

For type 0, native object selection computes this weight, using the game's
approximate square root for distance `d`:

```
weight = max(RGB)/255 * intensity
         / (1 + falloff*d + 1.2*falloff*d*d)
```

This is a light-selection weight, not the complete per-pixel shader equation
or a shadow-visibility test. The separate native point-light setup uses:

```
diffuse RGB = RGB * renderIntensity / 255
attenuation = (1, falloff, 1.2*falloff)
range       = 9999
```

In particular, type-0 range is not taken from field 64.

## Header Ambient

Header bytes 8..79 contain the global light record; bytes 80..91 contain three
ambient floats. The native setter scales incoming ambient by
`0.16326530277729034`, **not `1/255`**. Header ambient is copied to world offset
`0x108`, combined componentwise by maximum with sector ambient at `0xE8`, and
converted to rounded packed ARGB.

Character shader setup subsequently normalizes the scaled ambient RGBA by
approximately `1/255` and uploads shader constant `c116`. For a normalized
authored ambient channel `a`, the verified generation mapping is:

```
header ambient = clamp(a, 0, 1) * 255
CPU ambient    = clamp(a, 0, 1) * 0.16326530277729034
```

Do not invert the native attenuation: that overcompensates by approximately
6.125 times.

## Directional Light and Direct-Baking Profile

An explicitly enabled sun/moon uses a type-2 global light record, a normalized
light-travel direction, and sector flag `0x20`. Native setup produces D3D
directional type 3; surface visibility rays point opposite to the stored travel
direction. Disabled global-light and ambient inputs remain zero.

The current direct-light generation profile combines shared-edge subdivision,
brush occlusion, point-light records, optional ambient/directional lighting,
four-sample oversampling, and angle-threshold normal smoothing. Both point-light
intensity fields are initialized from authored intensity; falloff follows the
native `worldLighting_CalcLightFalloff` equation with cutoff `0.2`.

Indirect illumination, full style/flag semantics, and per-sector light
assignment remain unresolved. Moving-object lights can cross walls in the
current single-sector output. The verified record and CPU contracts do not
establish complete GPU shading or full gameplay-frame behavior.

## Native Evidence

Addresses refer only to the matching debug `Ghost.exe` and `Ghost.map`:

| Routine/data | Address |
|--------------|---------|
| Sector static-light reader | `0x005F2B62` |
| `tObject::GetStaticLights` | `0x00427690` |
| `CalculateLightIntensity` | `0x004270E0` |
| `rlLight_SetAmbient` | `0x006022D0` |
| Ambient scale constant | `0x007AB8A8` |
| Character shader setup | `0x005FCB50` |

The sector reader stores the light count at sector offset `0x10` and the array
pointer at `0xE4`; object selection consumes that same array.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NIL vertices](./nil-vertices.md) | [Level relationships](./level-relationships.md)
