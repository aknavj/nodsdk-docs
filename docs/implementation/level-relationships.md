# Level Format Relationships

Loading a level requires resolving a chain of cross-format references:

```
  ┌─────────────────────────────────┐
  │  NIL (.nil) — level geometry    │
  │                                 │
  │  Contains:                      │
  │  • shader name table            │──────┐
  │  • sector vertices + indices    │      │
  │  • surface groups with          │      │
  │    shaderIdx into name table    │      │
  └─────────────────────────────────┘      │
                                           │ shader names
                                           ▼
  ┌─────────────────────────────────┐      ┌─────────────────────────────────┐
  │  NSA (.nsa) — material defs     │◀─────│  name lookup (case-insensitive) │
  │                                 │      └─────────────────────────────────┘
  │  Per material entry:            │
  │  • texture    → DDS filename    │──────┐
  │  • glow       → DDS filename    │──────┤
  │  • detail     → DDS filename    │──────┤
  │  • envmap     → DDS filename    │──────┤
  │  • texture2   → DDS filename    │──────┤
  │  • shaderType → Xbox shader ID  │      │
  │  • intensity  → float scalar    │      │
  │  • detailscale→ float scalar    │      │
  │  • pageflip   → int frame count │      │
  └─────────────────────────────────┘      │
                                           │ texture names
                                           ▼
  ┌─────────────────────────────────┐      ┌─────────────────────────────────┐
  │  DDS/TGA/XPR — texture data     │◀─────│  search 13 directories          │
  │                                 │      │  case-insensitive matching      │
  │  DDS: DXT1, DXT5, A8R8G8B8     │      └─────────────────────────────────┘
  │  TGA: BGR/BGRA, RLE or raw     │
  │  XPR: packed archive, multiple  │
  │       named textures per file   │
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  NSD (.nsd) — scene entities    │
  │                                 │
  │  Per entity:                    │
  │  • template name (modifier 0)   │
  │  • position  (3 floats, D3D)    │
  │  • rotation  (3 floats, euler)  │
  │  • model ref (resolved ID 4)    │──────┐
  │                                 │      │
  │  Example template names:        │      │
  │  • ghost — player spawn         │      │
  │  • trigger_region               │      │
  │  • camera_region                │      │
  │  • alarm_region                 │      │
  │  • script_object                │      │
  │  • (model entities)             │      │
  └─────────────────────────────────┘      │
                                           │ model name
                                           ▼
  ┌─────────────────────────────────┐      ┌─────────────────────────────────┐
  │  NOD (.nod) — entity model      │◀─────│  3D/Models/<name>.nod           │
  │                                 │      │  case-insensitive lookup        │
  │  Same format as standalone NOD  │      └─────────────────────────────────┘
  │  Mesh groups reference NSA mats │──── NSA ──── DDS (same pipeline)
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  NPD (.npd) — navigation        │
  │                                 │
  │  Per record (26 bytes):         │
  │  • waypointId (uint16)          │
  │  • position (3 floats, D3D)     │
  │  • flags    (uint16)            │
  │  • component + outgoing range   │
  │  Directed links: 6 bytes each   │
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  NLFX (.nlfx) — effect list     │
  │                                 │
  │  One effect name per line       │──────┐
  └─────────────────────────────────┘      │
                                           │ name lookup
                                           ▼
  ┌─────────────────────────────────┐
  │  NFX (.nfx) — effect database   │
  │                                 │
  │  controller blocks matching     │
  │  names from NLFX                │
  └─────────────────────────────────┘
```

## Native Relationships and Boundaries

- **NIL:** vertex pools, static-light arrays, and Havok bodies are distinct
  inputs. Boundary planes define sector membership, not portal adjacency.
  Separate [lighting](./nil-lighting.md) and [collision](./nil-collision.md)
  guides describe their runtime contracts; no separate lightmap atlas has been
  established.
- **NSD + NOT:** resolve the complete ordered modifier stream through
  `Global.not` and scene-listed archives. Modifier 20 determines native type;
  modifier 4 supplies the model after [template expansion](./templates.md).
  Hardcoded template-name aliases, underscore stripping, and fixed-offset
  secondary scans are not native model resolution.
- **NPD:** nodes use world-space XYZ, not a rescaled navigation space.
  Navigation IDs and record-index links are independent of NSD placement IDs.
  Native sector lookup recomputes each waypoint's sector from its position.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NSA](../formats/nsa.md) | [DDS](../formats/dds.md) | [TGA](../formats/tga.md) | [XPR](../formats/xpr.md) | [NSD](../formats/nsd.md) | [NOT](../formats/not.md) | [NOD](../formats/nod.md) | [NPD](../formats/npd.md) | [NLFX](../formats/nlfx.md) | [NFX](../formats/nfx.md) | [Dependency graph](./dependency-graph.md)
