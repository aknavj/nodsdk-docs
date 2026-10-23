# NIL Collision and Spatial Data

The verified no-terrain [NIL v35 layout](../formats/nil.md) contains Havok mesh
and MOPP segments after sector records, followed by render-group collision
mappings. These bytes are not a separate lightmap atlas.

## Bodies, Ownership, and Bounds

A body carries float3 positions, uint16 triangle-list indices, eight-byte
material records, `moppInfo`, MOPP bytecode, and owner type/index. Owner types
are 0=world, 1=sector, and 2=terrain. In the eight examined originals, each
collision-present sector resolves to one sector-owned body.

Sector boundary planes and physics bounds serve different purposes. Sector
membership requires `dot(n, p) >= d` for every inward plane; a sector with zero
planes covers all space. Physics bounds must finitely and strictly enclose
generated collision geometry, with finite float spans. They do not restrict
membership in a zero-plane sector.

## Surface and Sound Lookups

Render-group mappings and material records use different interval conventions:

| Data | Meaning | Native lookup |
|------|---------|---------------|
| Group collision offset | Inclusive triangle-key start | Scan groups backwards; take last mapped start `<= key` |
| Material limit | Exclusive triangle-key end | Scan materials forwards; take first limit `> key` |

Equal group starts are allowed; the later group wins. A key below all mapped
starts has no surface group. The loader sets runtime group flag `0x02` when
the mapping is absent and clears it when present. Offsets are triangle keys,
not byte offsets or index-buffer offsets.

Each eight-byte material record contains:

| Byte | Size | Meaning |
|------|------|---------|
| 0 | 4 | Exclusive triangle-key limit |
| 4 | 2 | Sound type |
| 6 | 2 | Opaque bytes; preserve original nonzero values |

No matching material record means no sound override. Original mapped starts
and material limits are ordered and no greater than the owning body's triangle
count. Collision triangle order must remain stable when constructing render
groups so these lookups continue to identify the intended surfaces.

## Bounded MOPP Generation

The verified generator emits a deterministic balanced tree for one mesh
subpart, up to 65,535 vertices and 4,096 triangles. These are implementation
limits, not universal NIL or Havok format limits.

Input triangle indices remain primitive keys; the second shape-key word is
subpart zero. The generated instruction subset is:

| Opcodes | Meaning |
|---------|---------|
| `0x10..0x12` | Byte-axis splits |
| `0x23..0x25` | Two-offset 16-bit splits |
| `0x26..0x28` | Axis bounds |
| `0x30..0x51` | Triangle terminals |

Multi-byte bytecode operands are **big-endian**, despite the enclosing NIL's
little-endian scalar fields. `moppInfo.xyz` is the world offset;
`moppInfo.w` converts to 16.16 fixed point. Bounds include one quantization
cell of slack. This generation subset is not an arbitrary-original-MOPP decoder.

Reject invalid, degenerate, unrepresentable, or out-of-profile geometry without
publishing partial output. Regenerate MOPP from the collision mesh and reject
code that does not match the mesh.

## Geometry Generation Contract

The reconstructed classic-MAP path builds bounded brush half-spaces, subtracts
other brush interiors, removes opposing shared faces, and retains the earliest
brush's material/texture fields for same-facing coplanar overlaps. Fragments
retain their source face. Seam welding, precise T-junction insertion, and ear
triangulation preserve collinear boundary vertices.

Input solids require closed manifold boundaries and positive volume. The union
mesh requires balanced directed edges; edge/vertex-only solid contacts are not
filled. Binary32 collapse and work-limit overflow fail explicitly.

Separate per-surface render vertices preserve normals and texture seams while
retaining collision triangle order. Classic UV-axis selection uses the source
plane's binary32 cross product followed by double normalization, not a derived
CSG normal. Split vertices retain the source face's projection.

The bounded complete-world profile uses one primary-pool sector, observed sector
flags 5, no boundary planes or environment references, one sector-owned body, and
exact group mappings. Observed flags and opaque values do not establish complete
flag semantics or arbitrary multi-sector visibility behavior. A render-only sector
record is not a complete, collision-valid NIL.

## Native Evidence and Scope

In the matching debug executable, group lookup is
`tSector::FindSurfaceGroupWithTri` (`0x0052ECB0`) and sound lookup is
`hkNSIMeshShape::GetSoundType` (`0x004504E0`). Accelerated query paths include
AABB (`0x0063AAE0`), sphere (`0x0063A0D0`), and long ray (`0x0063BD80`);
the unaccelerated mesh ray is at `0x0062F520`.
Addresses are build-specific. Loader framing and controlled native queries do
not establish player/controller behavior, arbitrary dynamics, or complete game
loading.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NIL vertices](./nil-vertices.md) | [Level relationships](./level-relationships.md)
