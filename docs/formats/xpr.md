# XPR — Xbox Packed Resource

**Extension:** `.xpr`
**Type:** Binary
**Magic:** `0x30525058` ("XPR0")

Xbox-specific packed texture archive. Contains multiple named texture resources, each with a 20-byte D3D descriptor that encodes format, dimensions, and mipmap info in Xbox GPU-native format.

## File Layout

```
[Header: 16 bytes]
  uint32 magic         "XPR0"
  uint32 totalSize     Total file size
  uint32 headerSize    Offset to pixel data
  uint32 resourceCount Number of resources

[Resource Table: N × 8 bytes]
  uint32 nameOffset    Offset to null-terminated name string
  uint32 descriptorOff Offset to 20-byte D3D texture descriptor

[Name Strings: variable]
  Null-terminated ASCII strings

[Descriptors: N × 20 bytes]
  Opaque Xbox D3D texture descriptor

[Texture Data: bulk]
  Raw pixel data (DXT1/DXT5/A8R8G8B8)
```

The D3D descriptor encodes: format (DXT1/DXT3/DXT5/A8R8G8B8/L8/A8L8/etc.), width, height, mipmap count, and data offset within the texture data section.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [DDS](./dds.md) | [NSA](./nsa.md) | [Level Format Relationships](../implementation/level-relationships.md)
