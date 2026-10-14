# NSD — Nihilistic Scene Data

**Extension:** `.nsd`
**Type:** Binary

Scene entity database for a level. Contains entity type definitions and their
instance data (positions, rotations, properties). This is the **authoritative
source** for entity world-space positions. Used to extract entity spawn points
including the player "ghost" entity.

## Header (92 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 4 | magic | `NSD\x10` (0x1044534E) |
| 0x04 | 4 | entityCount | Header entity count (cutscene entities?) |
| 0x08 | 72 | metadata | Level metadata / ambient data |
| 0x50 | 4 | totalEntities | Total entity count from scanner (at offset 80) |
| 0x54 | 4 | sectorCount | Number of sectors/rooms (at offset 84) |
| 0x58 | 4 | reserved | Zero padding |

## Entity Block Layout

Entity blocks begin at offset 92 and are variable-length. Each entity uses a
tag-based property system with 36-byte property slots:

```
[Type Name: 32 bytes]
  Null-terminated ASCII string, zero-padded to 32 bytes.

[Property Block 0 (rotation): 36 bytes at T+32]
  uint32  tag1        = 1 (rotation tag)
  float[3] rotation   Euler angles (pitch, yaw, roll in degrees)
  byte[20] filler     Leftover name bytes / padding

[Property Block 1 (position): 36 bytes at T+68]
  uint32  tag2        = 2 (position tag)
  float[3] position   World-space position (D3D Z-up coordinates)
  byte[20] filler     Leftover name bytes / padding

[Property Block 2 (sector): 36 bytes at T+104]
  uint32  tag3        = 3 (model entity) or 15 (region entity)
  uint32  sectorId    Room/sector index (groups entities by area)
  float[2] posYZ      Duplicate of position Y, Z (stale data)
  byte[20] filler     Leftover name bytes / padding

[Property Block 3+ (optional): 36 bytes at T+140]
  uint32  tag4        = 4 (model name) or 0x2E (46, script ref) etc.
  char[]  value       Model filename or script name (null-terminated)
  ...                 Additional property blocks follow in 36-byte steps
```

## Tag Values

| Tag | Meaning | Data |
|-----|---------|------|
| 1 | Rotation | 3 floats: pitch, yaw, roll (degrees) |
| 2 | Position | 3 floats: X, Y, Z (D3D world-space) |
| 3 | Model entity | sector ID (u32), then stale position data |
| 4 | Model name | Null-terminated string (e.g., "door.nod") |
| 15 | Region class | Region class name (e.g., "cRegionSound") |
| 46 (0x2E) | Script ref | Script/animation name string |

## Entity Type Categories

Entity `typeName` values classify entities:
- **Model entities** (tag3=3): `instance_nocfile`, `ge_door*`, `movetrack_*`, `movenone_*`, etc.
- **Region entities** (tag3=15): `trigger_region`, `camera_region`, `alarm_region`, `norain_region`
- **Character entities**: `ghost` (player), `scvdriver*`, `infantry_hauler*`, `hydralisk*`, etc.
- **Pickup entities**: `powhealth`, `powenergy`, `powgrenade`, `powdata`, etc.
- **Skip types** (non-visual): `script_object`, `halo_*`, `cone_*`, `sparkfountain`, etc.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOT](./not.md) | [NOD](./nod.md) | [NIL](./nil.md) | [Level Format Relationships](../implementation/level-relationships.md) | [NOT Template Hierarchy](../implementation/templates.md) | [Coordinate system](../reference/coordinate-system.md)
