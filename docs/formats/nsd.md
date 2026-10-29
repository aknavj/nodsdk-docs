# NSD — Nihilistic Scene Data

**Extension:** `.nsd`
**Type:** Binary

Scene entity database for a level. Contains entity type definitions and their
instance data (positions, rotations, properties). This is the **authoritative
source** for entity world-space positions. Used to extract entity spawn points
including the player "ghost" entity.

## Prefix and First Object

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 4 | magic | `NSD\x10` (0x1044534E) |
| 0x04 | 4 | version | Supplied scenes use version 11 |
| 0x08 | 64 | metadata | Two 32-byte fields; semantics not fully reconstructed |
| 0x48 | 4 | flags | Scene flags |
| 0x4C | 4 | numTemplateFiles | Count of following 32-byte NOT archive names |
| 0x50 + 32*N | 4 | objectCount | Declared object count after the archive list |
| 0x54 + 32*N | 4 | modifierCount | First object's modifier count, not a sector count |
| 0x58 + 32*N | 4 | modifierID | First modifier ID, normally 0 (`basedon`) |
| 0x5C + 32*N | 32 | templateName | First `basedon` payload |

`N` is `numTemplateFiles`. All eight examined scenes have no additional archive
names, placing the first template-name payload at byte 92. This is **not a
fixed 92-byte header or the beginning of an object record**.

Archive names are nonempty, NUL-terminated ASCII fields of 32 bytes, in source
order. They name NOT archives, not placements or sky models. The verified native
scene layout has eight archive handles at `0x160..0x17C`, before the
object-pointer array at `0x180` and world pointer at `0x184`. New output is
limited to eight additional archives. The legacy scanner accepts nine; that
parser allowance must not be treated as a native runtime capacity.

## Entity Block Layout

Each object begins with a `uint32` modifier count followed by that many
**36-byte modifiers**, shared with NOT: `uint32 modifierID + byte[32] payload`.
The count includes the `basedon` modifier. Resolve by ID and source order, not
by assuming a fixed position for a property.

For a common object whose first modifiers are `basedon`, orientation, and
position, let `T` be the first template-name payload. These offsets illustrate
that sequence, not a mandatory property order:

```
[Modifier Count: uint32 at T-8]
[First Modifier ID: uint32 at T-4, normally 0]
[Template Name: 32 bytes at T]
  Null-terminated ASCII string, zero-padded to 32 bytes.

[Property Block 0 (rotation): 36 bytes at T+32]
  uint32  tag1        = 1 (rotation tag)
  float[3] rotation   Euler angles (pitch, yaw, roll in degrees)
  byte[20] filler     Leftover name bytes / padding

[Property Block 1 (position): 36 bytes at T+68]
  uint32  tag2        = 2 (position tag)
  float[3] position   World-space position (D3D Z-up coordinates)
  byte[20] filler     Leftover name bytes / padding

[Property Block 2: 36 bytes at T+104 in common records]
  uint32  tag         Often 3 (sector) or 15 (script)
  byte[32] payload    Interpret according to modifier ID

[Property Block 3+ (optional): 36 bytes at T+140]
  uint32  tag         E.g. 4 (model), 46 (CASTID), 161 (EFFECT1)
  byte[32] payload    Fixed width, including scalar/vector values
  ...                 Additional property blocks follow in 36-byte steps
```

## Tag Values

| Tag | Meaning | Data |
|-----|---------|------|
| 0 | basedon | Case-insensitive NOT template reference |
| 1 | Rotation | 3 floats: pitch, yaw, roll (degrees) |
| 2 | Position | 3 floats: X, Y, Z (D3D world-space) |
| 3 | sector | Sector ID (u32); remaining payload bytes are not extra properties |
| 4 | Model name | Null-terminated string (e.g., "door.nod") |
| 15 | script | Script class/reference; often used by regions |
| 20 | type | Runtime thing-type enum, distinct from template name |
| 46 (0x2E) | CASTID | String, not the script modifier |
| 47 | DESCRIPTIONID | String, not root/template flags |
| 143 | MODELSCALE | Float32 |
| 161 (0xA1) | EFFECT1 | String, not the script modifier |
| 174 | MODEL_LODSCALE | LOD selection scale, not geometry scale |
| 175 | MODELOFFSET | Three float32 components |

The legacy field `type_name` is a **template name**. Expand its modifiers to
obtain runtime type, model, scripts, and flags. Template-name patterns alone
must not determine model aliases, visibility, or runtime class.

## Bounded Point-Scene Output

The reconstructed point-scene writer emits version 11, empty metadata, zero
flags, an ordered archive list, and an object count. After each modifier array
it writes this 20-byte empty tail:

| Bytes | Field | Value |
|-------|-------|-------|
| 4 | motion-frame count | 0 |
| 4 | script-parameter count | 0 |
| 4 | inline-model-present flag | 0 |
| 4 | sector-link count | 0 |
| 2 | camera-region-present flag | 0 |
| 2 | prelighting count | 0 |

After all objects, the bounded writer emits this 192-byte trailer. Offsets are
relative to the trailer start, not the file start:

| Offset | Size | Bounded output |
|--------|------|----------------|
| 0 | 4 | Zero sound count |
| 4 | 12 | Three zero auxiliary world-array counts |
| 16 | 32 | Music reference, empty by default |
| 48 | 32 | Ambient-loop reference, empty by default |
| 80 | 4 | `soundEffects`, uint32; unexposed, zero |
| 84 | 4 | `musicVolume`, uint32; unexposed, zero |
| 88 | 4 | `ambientVolume`, uint32; unexposed, zero |
| 92 | 4 | Native reverb preset ID, uint32 in `0..25`; default 0 |
| 96 | 4 | Fog mode (`uint32`): 0 = none, 1 = linear |
| 100 | 12 | Fog color, three float32 RGB components in `0..255` |
| 112 | 4 | Linear fog start distance, float32 |
| 116 | 4 | Linear fog end distance, float32 |
| 120 | 4 | Camera far-clip distance, float32; 0 selects native default |
| 124 | 4 | `effectsFlags`, uint32; unexposed, zero |
| 128 | 32 | Sky-model reference, empty by default |
| 160 | 32 | Sky-animation reference, empty by default |

All four reference fields are 32-byte, NUL-terminated, zero-padded ASCII
strings. The numeric field names and unsigned 32-bit types at offsets 80,
84, 88, and 124 are confirmed by the executable-matched PDB's `tScene`
definition. Their trailer-to-scene mappings are `80 -> 0x100`,
`84 -> 0x124`, `88 -> 0x148`, and `124 -> 0xA8`. Native-reader probes with
distinct uint32 tags verify storage and complete stream consumption, not
valid authoring values. Volume units/ranges, sound-effects values and
effects-flag bits remain unresolved; do not reinterpret these words as floats.

See [NSD scene environment](../implementation/scene-environment.md) for
`MAPtoNSD` worldspawn settings, accepted values, defaults, and runtime limits.

Zero scene flags omit the optional `0x08000000` section. Sky-model references
are separate from NIL surface shader bindings.

Supported generated placement types are 0, 2, 7, and 8. Native type 8 is
`tContainer`, **not a brush-region record**. In the matching debug executable,
`gameThing_New` (`0x0052FE10`) allocates `0x3C0` bytes and invokes the container
constructor at `0x00518380`; the constructed object has vtable `0x0078E538` and
type 8 at object offset `0x10`. These addresses identify that executable only.

The verified scene-reader framing does not establish complete gameplay
initialization, player/controller construction, resource loading, or container
inventory behavior. This bounded output is not a general NSD serializer.

NSD placement coordinates do not override native
[player-placement routing](../implementation/level-relationships.md#player-placement-and-sector-lookup).

## Inline Region Geometry

After a counted region's modifiers, the verified layout includes:

1. A `uint32` motion-frame count and `count × 28` bytes of frames.
2. A `uint32` script-parameter count. Each parameter starts with a four-byte type:
   types 1–5 and 8–11 have four-byte payloads, type 6 has none, and type 7 has a
   `uint32` byte length followed by that many string bytes.
3. A `uint32` inline-model-present flag. A nonzero flag is followed by a standard
   92-byte NOD header and shader/bone/vertex/index/group streams.

Compute a region's local AABB from vertices referenced by valid LOD-0 triangles,
accounting for per-set bases, split TBN streams, triangle lists, degenerate
connectors, and strip restarts. Header bounds and unreferenced vertices do not
define this computed box. Native MatrixPYR rotation and entity position produce
its world-space bounding envelope.

This framing was checked against the original readers and 286 regions from
seven complete level triplets. It does not fully decode trailing region data,
all marker representations, or every subsequent scene section. Unknown layouts
must be reported as unavailable rather than located by plausible-float scans.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOT](./not.md) | [NOD](./nod.md) | [NIL](./nil.md) | [Level Format Relationships](../implementation/level-relationships.md) | [NSD Scene Environment](../implementation/scene-environment.md) | [NOT Template Hierarchy](../implementation/templates.md) | [Coordinate system](../reference/coordinate-system.md)
