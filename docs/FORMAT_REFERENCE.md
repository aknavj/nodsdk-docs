# StarCraft: Ghost — Asset Format Reference

> Reverse-engineered binary and text formats from the unreleased Xbox build of StarCraft: Ghost (Nihilistic/Swingin' Ape Studios, ~2004).
> All binary formats are **little-endian**. Coordinate system is **D3D left-handed Z-up** (X-right, Y-forward, Z-up).

Each format has its own page. Use the implementation guides for cross-format relationships and runtime data handling.

## Formats

| # | Format | Extension(s) | Type | Description |
|---|--------|-------------|------|-------------|
| 1 | [DDS](./formats/dds.md) | `.dds` | Binary | Standard DirectDraw Surface textures |
| 2 | [TGA](./formats/tga.md) | `.tga` | Binary | Targa images (uncompressed + RLE) |
| 3 | [XPR](./formats/xpr.md) | `.xpr` | Binary | Xbox packed texture archives |
| 4 | [NAD](./formats/nad.md) | `.nad` | Binary | Skeletal animation clips |
| 5 | [NOD](./formats/nod.md) | `.nod` | Binary | 3D models with skeleton + materials |
| 6 | [NMS](./formats/nms.md) | `.nms` | Text | Motion name → animation file mapping |
| 7 | [NRT/NUT](./formats/nrt-nut.md) | `.nrt` `.nut` | Binary | Terrain heightmap/patch data |
| 8 | [NIL](./formats/nil.md) | `.nil` | Binary | Indoor level geometry + portals + lightmaps |
| 9 | [NLFX](./formats/nlfx.md) | `.nlfx` | Text | Per-level effect reference list |
| 10 | [NPD](./formats/npd.md) | `.npd` | Binary | AI navigation/patrol waypoint graph |
| 11 | [NSA](./formats/nsa.md) | `.nsa` | Text | Material/shader property definitions |
| 12 | [NSD](./formats/nsd.md) | `.nsd` | Binary | Scene entity type definitions |
| 13 | [NLT](./formats/nlt.md) | `.nlt` | Text | Raw localization text |
| 14 | [NOC](./formats/noc.md) | `.noc` | Binary | Per-model collision meshes |
| 15 | [NLS/NLU](./formats/nls-nlu.md) | `.nls` `.nlu` | Text | Key-value localization strings |
| 16 | [NOT](./formats/not.md) | `.not` | Binary | Entity/object template database |
| 17 | [NFX](./formats/nfx.md) | `.nfx` | Text | Particle/VFX controller definitions |
| 18 | [NUI](./formats/nui.md) | `.nui` | Text | UI screen layout definitions |
| 19 | [NAK](./formats/nak.md) | `.nak` | Text | Camera animation keyframes |
| 20 | [NCE](./formats/nce.md) | `.nce` | Binary | Cutscene event data |
| 21 | [NCS](./formats/ncs.md) | `.ncs` | Binary | Cutscene shot/camera scripts |
| 22 | [NMB](./formats/nmb.md) | `.nmb` | Binary | Multi-LOD model bundles |
| 23 | [NNB](./formats/nnb.md) | `.nnb` | Binary | Animation bundles (paired with NMB) |

## Shared Reference

- [Coordinate System](./reference/coordinate-system.md)
- [Xbox Shader Types](./reference/xbox-shader-types.md)
- [Asset File Counts](./reference/asset-file-counts.md)

## Related Documentation

- [Implementation guides](./IMPLEMENTATION.md)
- [Documentation home](../README.md)
