# NOT — Nihilistic Object Templates

**Extension:** `.not`
**Type:** Binary
**Magic:** `0x10544F4E` ("NOT\x10")
**Version:** 1

Entity template database defining the property hierarchy for all game object types. Contains ~1224 templates organized in an inheritance tree rooted at 8 base classes.

## File Layout

```
[Header: 12 bytes]
  uint32  magic          "NOT\x10"
  uint32  version        Always 1
  uint32  numTemplates   Template count

[Table of Contents: numTemplates × 36 bytes]
  char[32] name          Template name
  uint32   dataOffset    Offset to template data

[Template Data: variable]
  Per template:
    uint32  flags         47=root, 0=child
    char[32] parent       Parent template name
    uint32  numProperties Property count

    [Properties: variable]
      Three encoding types:
        Standard (36 bytes): uint32 propID + uint8[32] value
        Boolean  (5 bytes):  uint32 propID(=1) + uint8 value
        Comment  (variable): uint32 strLen + char[] text
```

## Root Templates (flags=47)

| Name | Description |
|------|-------------|
| `_ACTOR` | Characters, NPCs, enemies |
| `_EFFECT` | Visual/audio effects |
| `_PROJECTILE` | Weapons projectiles |
| `_PROP` | Static/dynamic props |
| `_REGION` | Trigger regions/volumes |
| `_SENSOR` | Detection sensors |
| `_UIELEMENT` | HUD/UI elements |

All other templates inherit from one of these roots.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NSD](./nsd.md) | [NOT Template Hierarchy](../implementation/templates.md)
