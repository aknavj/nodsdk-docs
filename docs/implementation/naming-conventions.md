# Naming Conventions That Bind Formats

## Same-Stem Pairing (Level Files)

Level assets share a common filename stem. Given a level `1_2_1a_Miners_Bunker`:

```
Levels/
  1_2_1a_Miners_Bunker.nil      geometry + portals + lightmaps
  1_2_1a_Miners_Bunker.nsd      entity instances (types, positions, rotations)
  1_2_1a_Miners_Bunker.npd      entity placement records
  1_2_1a_Miners_Bunker.nlfx     effect name references

Materials/
  1_2_1a_Miners_Bunker.nsa      material definitions for this level
  1_2_1a_Miners_Bunker.xpr      packed texture archive for this level
```

All six files derive from the same stem. The `.nil` file is the entry point — companion files are found by extension substitution.

## Same-Stem Pairing (Model Files)

Model bundles and animation bundles share stems:

```
3D/Models/
  <name>.nod      standalone model
  <name>.nmb      model bundle (multiple LODs)

3D/Collision/
  <name>.noc      collision mesh

3D/Anims/
  <name>.nnb      animation bundle (paired with .nmb)
  <name>.nad      standalone animation clip
```

## Same-Stem Pairing (Cutscene Files)

Cutscene event/script pairs share stems:

```
Cutscenes/
  <name>.nce      cutscene events
  <name>.ncs      cutscene camera/shot script
```

## NSA Material Name → Texture Path

NSA materials reference textures by name (no path, no extension). The name must be resolved against the directory tree:

```
Materials/Common/          shared textures
Materials/Metal/           metal surface textures
Materials/Stone/           stone surface textures
Materials/Wood/            wood surface textures
Materials/Organic/         organic surface textures
Materials/Glass/           glass/transparent textures
Materials/Effects/         VFX textures
Materials/Terrain/         terrain blend textures
Materials/Colors/          solid color textures
Materials/Lights/          light cookie textures
Materials/Shaders/         shader-specific textures
Materials/                 level-specific textures
3D/Materials/              model-specific textures
```

Resolution is case-insensitive. The `.dds` extension is implied. NSA texture name normalization strips `.dds` suffixes and trailing whitespace-separated tokens (e.g. `"rock_wall normals"` → `"rock_wall"`).

## NIL/NOD Shader Name → NSA Material

NIL surface groups and NOD mesh groups store material names as fixed-width strings (32 or 33 bytes, null-padded). These names are keys into the NSA material database, matched case-insensitively.

## NSD Entity Type → NOD Model

NSD entity records contain an entity type name string. For model-type entities, a secondary scan at offset T+140 locates a `.nod` model filename in 36-byte blocks. The model path is constructed as `3D/Models/<name>.nod` and resolved case-insensitively or by fuzzy matching (stripping underscores).

## NMS Motion Name → NAD Path

NMS maps gameplay state names to `.nad` animation file paths via tab-delimited text entries.

## NLFX Effect Name → NFX Controller

NLFX lists effect names (one per line) that reference controller blocks defined in the global `game.nfx` file.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NIL](../formats/nil.md) | [NOD](../formats/nod.md) | [NMB](../formats/nmb.md) | [NNB](../formats/nnb.md) | [NOC](../formats/noc.md) | [NCE](../formats/nce.md) | [NCS](../formats/ncs.md) | [NSA](../formats/nsa.md) | [NSD](../formats/nsd.md) | [NMS](../formats/nms.md) | [NLFX](../formats/nlfx.md) | [Dependency graph](./dependency-graph.md)
