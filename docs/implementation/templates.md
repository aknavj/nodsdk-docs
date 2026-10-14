# NOT Template Hierarchy

NOT organizes ~1224 entity templates in an inheritance tree:

```
  Root templates (flags=47):
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ _ACTOR   │  │ _EFFECT  │  │_PROJECTILE│  │  _PROP   │
  └────┬─────┘  └────┬─────┘  └────┬──────┘  └────┬─────┘
       │             │             │               │
       ├─ Marine     ├─ Explosion  ├─ Bullet       ├─ Crate
       ├─ Zergling   ├─ Sparks     ├─ Missile      ├─ Door
       ├─ Firebat    └─ ...        └─ ...          └─ ...
       └─ ...

  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ _REGION  │  │ _SENSOR  │  │_UIELEMENT│
  └────┬─────┘  └────┬─────┘  └────┬─────┘
       │             │             │
       ├─ trigger_   ├─ motion_    ├─ healthbar
       ├─ camera_    ├─ sound_     ├─ ammo_count
       └─ ...        └─ ...        └─ ...

  Each child template:
  • parent name (32 bytes) — inherits all parent properties
  • numProperties — additional/overridden properties
  • Properties: ID + 32-byte value, or boolean (5B), or comment string
```

Template names in NOT match entity type names found in NSD scene data — this is how the engine resolves entity capabilities and default property values.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NOT](../formats/not.md) | [NSD](../formats/nsd.md) | [Dependency graph](./dependency-graph.md)
