# TGA — Targa Image

**Extension:** `.tga`
**Type:** Binary (standard TGA format)
**Magic:** None (no signature; validated via header fields)

Standard Targa format. Supports uncompressed and RLE-compressed true-color images.

## Header (18 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 1 | idLength | Image ID field length |
| 0x01 | 1 | colorMapType | 0=none, 1=present |
| 0x02 | 1 | imageType | 2=uncompressed, 10=RLE |
| 0x03 | 5 | colorMapSpec | First entry, length, bit depth |
| 0x08 | 10 | imageSpec | Origin X/Y, width, height, bpp, descriptor |

**Supported image types:** Uncompressed RGB (type 2), RLE RGB (type 10).
**Bit depths:** 24-bit (BGR) and 32-bit (BGRA).

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [DDS](./dds.md) | [XPR](./xpr.md)
