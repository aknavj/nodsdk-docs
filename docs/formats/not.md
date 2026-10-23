# NOT — Nihilistic Object Templates

**Extension:** `.not`
**Type:** Binary
**Magic:** `0x10544F4E` ("NOT\x10")
**Version:** 1

Object-template archive. `Global.not` contains 1,224 templates, but inheritance
is **ordered modifier expansion**, not a fixed single-parent tree.

## File Layout

```
[Header: 12 bytes]
  uint32  magic          "NOT\x10"
  uint32  version        Always 1
  uint32  numTemplates   Template count

[Table of Contents: numTemplates × 36 bytes]
  char[32] name          Template name
  uint32   dataOffset    Offset to template data

[Template Data: variable]
  Per template:
    uint32  numModifiers
    [Modifiers: numModifiers × 36 bytes]
      uint32   modifierID
      byte[32] payload
    uint32  editorDataLength
    byte[]  editorData        editorDataLength bytes
```

Editor comments and metadata belong to the separate length-prefixed editor
block. They are not alternate runtime modifier encodings. Preserve unknown
editor bytes and archive-tail bytes: the supplied archive has 36 additional,
loader-unconsumed bytes after the last `scourge_track` record. They must not be
invented into another property.

## Runtime Modifier IDs

| ID | Name | Payload |
|----|------|---------|
| 0 | basedon | Template-name string |
| 1 | orientation | Float3 Euler angles in degrees; still a 32-byte payload |
| 2 | position | World-space float3 |
| 3 | sector | Integer bits |
| 4 | model | Model-name string |
| 12 | renderflags | Integer bits; bit `0x2000` enables model offset |
| 15 | script | Script class/reference string |
| 20 | type | Native runtime thing-type enum |
| 46 | CASTID | String |
| 47 | DESCRIPTIONID | String; **not root/template flags** |
| 143 | MODELSCALE | Float32 |
| 161 | EFFECT1 | String |
| 174 | MODEL_LODSCALE | Float32; LOD selection, not geometric scale |
| 175 | MODELOFFSET | Float3 |

IDs 0, 1, and 47 were previously misread as parent headers, five-byte booleans,
or root flags. They all use the same fixed-width runtime modifier layout.

## Archive Resolution and Expansion

The native startup opens `Misc/Templates/Global.not` first, followed by the
NSD-listed archives in source order, relative to `Misc/Templates/`. A missing
extension receives `.not`. Names and archive lookup are case-insensitive.

The runtime registry has 64 archive slots and a shared template-name hash:
**the first loaded duplicate name wins**. Reopening an archive does not register
its names again. Index all selected archives before expansion because a
reference can target a later-loaded archive.

Process modifiers in source order:

1. ID 0 expands the referenced template **at that position**.
2. Other IDs replace an existing same-ID entry in place, or append a new entry.
3. A later `basedon` can therefore overwrite an earlier local value.

The reconstructed resolver rejects missing references, cycles, and expansions
beyond the native 128-modifier capacity, without returning partial results.
Apply the same expansion to the complete NSD placement modifier stream.
A local model value does not bypass an earlier bad reference, and an explicitly
empty model remains empty.

See [ordered templates and static transforms](../implementation/templates.md)
for precedence examples and the order-sensitive model transform contract.

## Native Metadata and Evidence

The matching debug executable exposes 177 metadata records at `0x00A05250`,
each 40 bytes: 32-byte name, four-byte handler address, four-byte native type.
Handler words begin at `0x00A05270`. Verified native storage codes are
0=string, 2=float32, 3=integer bits, and 4=float3. ID 68 (`PARTICLENAME`) has the
unusual native type code 68 and must remain raw. Storage type alone does not
establish signedness, enum values, flag meanings, or gameplay behavior.

Relevant paths are archive/editor-data loading (`0x0068EFB0`),
counted modifier reading (`0x0068F680`), ordered replacement (`0x00543F60`),
case-insensitive lookup (`0x0068F520`), and NSD modifier/section reading
(`0x0053F2E0`).
These addresses apply only to the matching debug executable and symbol map.
The sampled scenes' single-`basedon`, unrepeated-ID records do not restrict the
general ordered grammar.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NSD](./nsd.md) | [Ordered Templates and Static Transforms](../implementation/templates.md)
