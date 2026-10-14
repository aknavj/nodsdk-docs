# DDS — DirectDraw Surface

**Extension:** `.dds`
**Type:** Binary (standard Microsoft DDS format)
**Magic:** `0x20534444` ("DDS ")

Standard DirectDraw Surface container used as the primary texture format. Supports DXT1 (BC1), DXT5 (BC3) compressed formats and uncompressed A8R8G8B8 (BGRA).

## Header (128 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 4 | magic | `0x20534444` ("DDS ") |
| 0x04 | 4 | size | Header size (124) |
| 0x08 | 4 | flags | Content flags |
| 0x0C | 4 | height | Texture height in pixels |
| 0x10 | 4 | width | Texture width in pixels |
| 0x14 | 4 | pitchOrLinearSize | Row pitch or compressed size |
| 0x18 | 4 | depth | Volume depth (usually 0) |
| 0x1C | 4 | mipMapCount | Number of mipmap levels |
| 0x20 | 44 | reserved | Unused |
| 0x4C | 4 | pixelFormatSize | Pixel format struct size (32) |
| 0x50 | 4 | pixelFormatFlags | Pixel format flags |
| 0x54 | 4 | fourCC | Compression format (`DXT1`, `DXT5`, etc.) |
| 0x58 | 4 | rgbBitCount | Bits per pixel (uncompressed) |
| 0x5C | 16 | bitMasks | R, G, B, A channel masks |
| 0x6C | 4 | caps1 | Surface complexity flags |
| 0x70 | 4 | caps2 | Additional surface flags |
| 0x74 | 12 | reserved2 | Unused |

Pixel data follows immediately after the header. Mipmap chain is stored consecutively, each level half the previous dimensions.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [TGA](./tga.md) | [XPR](./xpr.md) | [NSA](./nsa.md) | [Naming Conventions That Bind Formats](../implementation/naming-conventions.md)
