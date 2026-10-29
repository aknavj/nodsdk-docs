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

Scene-wide sky/audio references, fog, far clipping, and reverb are stored in the
NSD trailer, separately from entity modifiers and NIL surface data. See
[NSD scene environment](./scene-environment.md) for supported settings and
their runtime limits.

## Player Placement and Sector Lookup

NSD placement coordinates are authored world-space data, not a guarantee of
the final player transform. In the matching debug build,
`gameMultiplayer_PlacePlayer` (`0x0051F7D0`) selects `FindPlayerStart` when
placement flag `0x8` or `mainSave_bRestoring` is nonzero, rather than directly
testing multiplayer mode.

Without either condition, a successful current-position sector query preserves
the player transform; a failed query resets it to identity at the origin.
Player-start lookup also falls back to the origin if both primary and
zero-first-selector searches fail. NSD placement coordinates do not override
these routing rules.

Controlled checks execute original routing, lookup and transform-reset code
with spatial-query and actor/world lifecycle boundaries substituted. This is
build-specific evidence, not proof of player/controller physics or actual
teleportation. The separate [collision-start observations](./nil-collision.md#collision-starting-positions)
describe a native sphere probe, not the full player path.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NSA](../formats/nsa.md) | [DDS](../formats/dds.md) | [TGA](../formats/tga.md) | [XPR](../formats/xpr.md) | [NSD](../formats/nsd.md) | [NOT](../formats/not.md) | [NOD](../formats/nod.md) | [NPD](../formats/npd.md) | [NLFX](../formats/nlfx.md) | [NFX](../formats/nfx.md) | [NSD scene environment](./scene-environment.md) | [Dependency graph](./dependency-graph.md)
