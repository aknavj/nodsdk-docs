# Model Format Relationships

## NOD Internal References

```
  NOD (.nod) file contents:
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  Header                                          │
  │  • shaderCount, boneCount, vertexSetCount,       │
  │    meshGroupCount, indexCount                     │
  │  • boundsMin, boundsMax (AABB)                   │
  │                                                  │
  │  Shader Names ─────────────────────────────────┐ │
  │  • shaderCount × 32-byte null-padded strings   │ │
  │                                                │ │
  │  Bones                                         │ │
  │  • boneCount × 64-byte entries                 │ │
  │  • each: position, orientation, offset,        │ │
  │          parent index, joint ID                │ │
  │                                                │ │
  │  Vertex Sets                                   │ │
  │  • per set: count × stride raw vertex data     │ │
  │  • type 0: rigid     (32B per vert)            │ │
  │  • type 1: 1-bone    (36B per vert)            │ │
  │  • type 2: weighted  (48B per vert)            │ │
  │                                                │ │
  │  Index Buffer                                  │ │
  │  • indexCount × uint16                         │ │
  │                                                │ │
  │  Mesh Groups ──────────────────────────┐       │ │
  │  • meshGroupCount × 56-byte entries    │       │ │
  │  • shaderIndex ────────────────────────┼───────┘ │
  │  • strip/list index ranges into ───────┼─→ Index Buffer
  │  • baseVertex offset into ─────────────┼─→ Vertex Sets
  │  • vertexSetIndex ─────────────────────┼─→ Vertex Sets
  │  • aBones[20] bone palette ────────────┼─→ Bones
  │                                        │       │
  └────────────────────────────────────────┴───────┘
                    │
                    │ shaderIndex → shader name string
                    ▼
              NSA material lookup → DDS texture path
```

## NOD ↔ NAD Bone Mapping

NAD bone tracks target NOD bones by index:

```
  NOD skeleton              NAD tracks
  ┌──────────────┐          ┌──────────────────────┐
  │ bone[0] root │◀─────────│ track: boneNumber=0  │
  │ bone[1]      │◀─────────│ track: boneNumber=1  │
  │ bone[2]      │          │   (no track = static) │
  │ ...          │          │ ...                   │
  │ bone[42]     │◀─────────│ track: boneNumber=42 │
  └──────────────┘          └──────────────────────┘

  Track types:
  • TRACK_ROTATION (0) — euler angles (pitch/yaw/roll in degrees)
  • TRACK_POSITION (1) — translation offset

  Keyframe evaluation (cubic polynomial):
    v(t) = value + t·b + t²·c + t³·d
```

## NOD ↔ NOC Collision Pairing

Almost all NOD render models have a corresponding NOC file that defines the
collision geometry used by the physics engine.  The two files share the same
model name but live in different asset directories:

```
  3D/Models/<name>.nod        render mesh (detailed, textured)
  3D/Collision/<name>.noc     collision mesh (simplified, untextured)
```

NOC contains its own vertex/index data — a simplified, lower-poly version of
the NOD geometry purpose-built for physics.  There is no explicit link in the
file data; they are paired by filename convention at load time
(`AssetBuilder` strips the `.nod` extension and probes for a matching `.noc`).

**Entity collision is mesh-based, not primitive-based.**  Entity models use
their NOC trimesh for physics collision (`RigidBody::addTriMesh`), not
capsules or spheres.  The only capsule collider in the engine is the **player
character controller** (Nova), which is a dynamic body and does not have a
NOC file — it uses an ODE capsule for character movement and ground detection.

For the rare entities that lack a NOC file, the physics system falls back to
an axis-aligned box derived from the model's bounding box
(`RigidBody::addBox` with extents from `bbox_min`/`bbox_max`).

### Debug Visualization Mapping

| Toggle | Key | What it shows |
|--------|-----|---------------|
| **Collision debug** | `C` | NOC mesh wireframes — cyan (level NIL sectors), green (entity composite, world-space), magenta (per-entity, model-local) |
| **Physics debug** | `P` | Actual ODE physics shapes — capsules, boxes, trimesh AABBs, velocity vectors |

The collision debug (C) shows the **source collision geometry** from the asset
files.  The physics debug (P) shows the **runtime ODE shapes** that the
physics engine actually uses for simulation — these include the player capsule,
NOC trimeshes (rendered as ODE AABBs), and AABB-fallback boxes.

## NMB Bundle Structure

NMB packs multiple model entries into a single file, including collision members
whose `.noc` identities remain distinct from visible `.nod` models:

```
  NMB file:
  ┌─────────────────────────────────────────┐
  │ Header: version, modelCount, totalMem   │
  │                                         │
  │ Entry[0] ─────────────────────────────┐ │
  │  • compact header (5B)                │ │
  │  • name (32B)                         │ │
  │  • vertex sets  (same types as NOD)   │ │
  │  • index buffer                       │ │
  │  • shader names → NSA lookup          │ │
  │  • bones (26B each, compact NOD)      │ │
  │  • mesh groups + bone remap tables    │ │
  │  • markers                            │ │
  │  • optional v2 packed vertex colors    │ │
  │                                       │ │
  │ Entry[1] ─────────────────────────────┤ │
  │  • ...same structure...               │ │
  │                                       │ │
  │ Entry[N-1] ───────────────────────────┘ │
  └─────────────────────────────────────────┘
          │
          │ paired by filename stem
          ▼
  NNB file:
  ┌─────────────────────────────────────────┐
  │ Header: version, animCount, dataSize    │
  │                                         │
  │ Entry[0]: name (64B) + offset           │──→ .nad file reference
  │ Entry[1]: name (64B) + offset           │──→ .nad file reference
  │ ...                                     │
  └─────────────────────────────────────────┘
```

The NMB header's `totalMem` is the native 32-bit CPU object arena, not file
length or combined vertex/GPU-buffer size. Vertex, index, TBN, blend, and optional
color buffers are separate allocations. See the [NMB format](../formats/nmb.md)
for the verified arena formula, compact group mapping, and version-2 color flag.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NOD](../formats/nod.md) | [NAD](../formats/nad.md) | [NOC](../formats/noc.md) | [NMB](../formats/nmb.md) | [NNB](../formats/nnb.md) | [Dependency graph](./dependency-graph.md)
