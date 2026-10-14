# NSA — Nihilistic Shader Archive

**Extension:** `.nsa`
**Type:** Text (Quake 3 Arena shader style)

Material property definitions in a Quake 3 style text format. Defines texture references, blend modes, shader types, and rendering parameters for level and model materials.

## Format

```
MaterialName
{
    key1 value1
    key2 value2
    ...
}
```

Common properties include: `diffuseMap`, `glowMap`, `detailMap`, `envMap`, `shaderType`, `blendMode`, `alphaTest`, `intensity`, `detailScale`.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [DDS](./dds.md) | [XPR](./xpr.md) | [NOD](./nod.md) | [NIL](./nil.md) | [NSA Material Properties](../implementation/material-properties.md) | [Naming Conventions That Bind Formats](../implementation/naming-conventions.md) | [Coordinate system](../reference/coordinate-system.md)
