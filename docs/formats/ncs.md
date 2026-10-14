# NCS — Nihilistic Cutscene Script

**Extension:** `.ncs`
**Type:** Binary
**Version:** 1

Cutscene camera/shot scripts paired 1:1 with `.nce` event files. Contains actors, sound cues, and conversation triggers with structured records. 216 files in `Cutscenes/`. Validated: 216/216 files parse successfully.

## Header Variants

| variant1 | variant2 | variant3 | Count | Notes |
|----------|----------|----------|-------|-------|
| 1 | 4 | 0 | 118 | Most common |
| 1 | 0 | 0 | 56 | Camera-only (no actors) |
| 1 | 6 | 0 | 10 | |
| 0 | 0 | 0 | 8 | |
| 256 | 4 | 0 | 6 | |
| 1 | 7 | 0 | 4 | Includes embedded map names |
| *(other)* | | | 14 | Rare variants |

## File Layout

```
[Header: 24 bytes]
  +0   uint32   version        Always 1
  +4   uint16   variant1       Header sub-type field
  +6   uint16   variant2       Header sub-type field
  +8   uint16   variant3       Header sub-type field
  +10  uint16   shotCount      Number of camera shots (matches NCE shotCount)
  +12  uint16[3] reserved
  +18  uint16   eventCount     Number of scripted events (matches NCE eventCount)
  +20  uint16[2] reserved

[Camera Block: variable size, starts at byte 24]
  Base: ~94 bytes of camera config data (mostly zeros for NAD-driven cameras)
  Optional: 32-byte camera NAD path, validated by ".nad" suffix
  Camera block ends at first actor marker, or at end-of-file if no actors
  Sizes observed: 126 (15 files), 94, 178, 266, 270, 370, 406, 606, 686, 778, 2378

[Actor Records: variable, each preceded by 12-byte marker]
  Marker pattern: uint32(1), uint32(N), uint32(5) — N=1 (base only) or N=2 (has anim states)
  Validated by checking that byte at marker+12 is printable ASCII

  Per actor (140 bytes base):
    +0   uint32  marker1        Always 1
    +4   uint32  marker2        1 = base only, 2 = has anim state records
    +8   uint32  actorType      Always 5
    +12  char[32] modelName     Template model name
    +44  char[32] instanceName  Instance name
    +76  uint8[20] flags        Config flags (layout differs from NCE)
    +96  char[32] nadPath       Per-actor NAD path (position varies — see below)
    +128 uint8[12] tail         Fixed tail: uint32 unk, float(30.0), uint32 unk

  NAD Path Position (within actor record bytes 76–139):
    When uint32@+92 == 9999 (0x270F): NAD at +96, tail at +128
    When uint32@+92 == 0:             tail at +96, NAD at +108
    Parser scans for ".nad" suffix within bytes 76–140 for robustness

  31 of 216 files contain actors (95 total actors found)

[Anim State Records: after actor base, when marker2 == 2]
  +140 uint32  animStateCount  Total number of anim states (explicit count)
  Then animStateCount × 44-byte type-2 records:
    +0   uint32  type           Always 2
    +4   float   frameTime      Trigger frame
    +8   uint32  unknown        Reserved
    +12  char[32] name          State name

  9 total anim states across all files

[Sound Cue Block: type-3 header + structured records]
  Header (20 bytes):
    +0   uint32  type           Always 3
    +4   uint32  subType        Sub-type (typically 1)
    +8   uint32  reserved       0
    +12  uint32  cueCount       Number of sound cue records
    +16  uint32  reserved       0

  Per cue (76 bytes):
    +0   float   frameTime      Trigger frame (e.g. 1.1, 3.06, 8.26)
    +4   uint32  reserved       0
    +8   char[32] name          Sound cue name (e.g. "SF121ASCV42")
    +40  uint32  reserved       0
    +44  uint32  reserved       0
    +48  float   volume         Playback volume (typically 1.0)
    +52  float   fadeIn          Fade-in time (typically 0.1)
    +56  float   range          Audible range (typically 10.0)
    +60  uint8[16] padding      Zeros

[Conversation Block: type-2 header + name + frame time]
  Header (12 bytes):
    +0   uint32  type           Always 2
    +4   uint32  count          Number of conversations (typically 1)
    +8   uint32  reserved       0

  Per conversation (48 bytes):
    +0   char[32] name          Conversation ID (e.g. "cM121_EscortSCV")
    +32  uint8[12] config       Config/padding
    +44  float   frameTime      Trigger frame (e.g. 16.1)

  2 files contain conversation triggers
```

## NCS vs NCE Comparison

| Feature | NCE | NCS |
|---------|-----|-----|
| Header | Variable (version + mapName) | Fixed 24 bytes |
| Camera pos/target/FOV | Yes (111-byte block) | No (NAD-driven) |
| Actor marker | `u32(1),u32(1)` + model@+8 | `u32(1),u32(N),u32(5)` + model@+12 |
| Actor flags | 23 bytes | 20 bytes |
| Actor NAD offset | Fixed +95 | Variable (+96 or +108, scan for ".nad") |
| Anim state count | Implicit (animIndicator-1) | Explicit u32 at +140 |
| Sound cue frame times | Heuristic (often 0.0) | Structured (76-byte records) |
| Conversation frame times | Heuristic (often 0.0) | Structured (type-2 block) |

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NCE](./nce.md) | [NAK](./nak.md)
