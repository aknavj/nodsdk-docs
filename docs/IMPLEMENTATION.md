# Format Mapping & Dependencies

How StarCraft: Ghost asset formats reference each other, the naming conventions that bind them, and the data relationships between formats.

Each topic has its own guide. For binary layouts and text syntax, use the [format reference](./FORMAT_REFERENCE.md).

## Guides

| Topic | Coverage |
|-------|----------|
| [Cross-Format Dependency Graph](./implementation/dependency-graph.md) | Overview of references between asset formats. |
| [Naming Conventions That Bind Formats](./implementation/naming-conventions.md) | Filename pairing, material lookup, and asset path resolution. |
| [Level Format Relationships](./implementation/level-relationships.md) | How geometry, materials, entity placements, navigation, and effects fit together. |
| [NSD Scene Environment](./implementation/scene-environment.md) | Worldspawn settings for sky, audio references, fog, far clipping, and reverb. |
| [Model Format Relationships](./implementation/model-relationships.md) | Model internals, bone mapping, collision pairing, and bundles. |
| [NSA Material Properties](./implementation/material-properties.md) | Material properties, Xbox shader behavior, and blend modes. |
| [NIL Vertex Data Layout](./implementation/nil-vertices.md) | Level vertex layouts and baked lighting decoding. |
| [NIL Lighting](./implementation/nil-lighting.md) | Baked diffuse, moving-object lights, ambient mapping, and directional lighting. |
| [NIL Collision and Spatial Data](./implementation/nil-collision.md) | Havok body ownership, surface/sound mappings, bounded MOPP, and geometry generation. |
| [NOD Vertex Data Layouts](./implementation/nod-vertices.md) | Rigid, single-bone, and weighted vertices with bone palettes. |
| [NOD Index Buffer Encoding](./implementation/nod-indices.md) | Triangle strips, triangle lists, and shared index buffers. |
| [NAD Animation Data Model](./implementation/animation.md) | Cubic keyframe curves and bone hierarchy evaluation. |
| [Ordered Templates and Static Transforms](./implementation/templates.md) | Archive precedence, in-place modifier expansion, runtime type/model lookup, and transforms. |
| [Coordinate System](./implementation/coordinate-system.md) | Spatial axes and the data that uses them. |

[Documentation home](../README.md)
