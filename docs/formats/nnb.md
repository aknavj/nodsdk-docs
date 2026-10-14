# NNB — Nihilistic Animation Bundle

**Extension:** `.nnb`
**Type:** Binary
**Version:** 1

Animation bundle paired 1:1 with `.nmb` model bundles. Contains references to `.nad` animation files. 40 files in `3D/Anims/` (7 are empty with count=0).

## File Layout

```
[Header: 12 bytes]
  uint32  version      Always 1
  uint32  animCount    Number of animation entries
  uint32  dataSize     Total data size

[Entries: variable]
  Per entry at 32-byte aligned boundaries:
    char[64]  name     .nad filename
    size_t    offset   Byte offset in bundle
```

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NMB](./nmb.md) | [NAD](./nad.md) | [Model Format Relationships](../implementation/model-relationships.md) | [NAD Animation Data Model](../implementation/animation.md)
