# NCE — Nihilistic Cutscene Events

**Extension:** `.nce`
**Type:** Binary
**Version:** 1

Cutscene event data defining actors, animations, camera, sound cues, and conversation triggers for cinematics. 207 files in `Cutscenes/`, each paired 1:1 with a `.ncs` script file. Validated: 207/207 files parse successfully.

## File Layout

```
[Header]
  +0   uint32  version         Always 1
  +4   uint8   unknownFlag     Single flag byte
  +5   uint32  mapNameLen      Map name string length
  +9   char[]  mapName         Map name (mapNameLen bytes, null-padded)

[Timing Block: 40 bytes, at offset 9 + mapNameLen]
  +0   uint32  numFrames       Total frame count (e.g. 38)
  +4   float   timeScale       Playback time scale (1.0)
  +8   ...     reserved        18 bytes
  +26  uint16  shotCount       Number of camera shots
  +28  ...     reserved        6 bytes
  +34  uint16  eventCount      Number of scripted events

[Camera Base Block: 111 bytes]
  +0   ...     config          67 bytes of camera config flags
  +67  float   posX            Camera initial position X
  +71  float   posY            Camera initial position Y
  +75  float   posZ            Camera initial position Z
  +79  float   reserved        Unused (0)
  +83  float   fov             Field of view in degrees (typically 50–80)
  +87  float   reserved        Unused (0)
  +91  float   targetX         Camera look-at target X
  +95  float   targetY         Camera look-at target Y
  +99  float   targetZ         Camera look-at target Z
  +103 ...     padding         8 bytes

[Camera NAD: optional 32 bytes at camera+111]
  char[32] cameraNAD           External camera animation path (e.g. "1_2_1a_tankRush_camera.nad")
                               Validated by checking for ".nad" suffix (len > 8)
                               23 of 207 files have external camera NAD

[Actor Records: variable, each preceded by 8-byte marker]
  Marker pattern: uint32(1), uint32(1) — searched via memmem()
  Validated by checking that byte at marker+8 is printable ASCII (0x20..0x7E)

  Per actor (140 bytes base):
    +0   uint32  marker1        Always 1
    +4   uint32  marker2        Always 1
    +8   char[32] modelName     Template model name (e.g. "nova_cut")
    +40  char[32] instanceName  Instance name (e.g. "Nova_121a_tankRush")
    +72  uint8[23] flags        Flag bytes:
         [2] visibility         0x00=hidden, 0x01=visible, 0xFF=varies
         [3] animIndicator      (value-1) = number of extra anim state records
    +95  char[32] nadPath       Per-actor animation NAD path
    +127 uint8[13] tail         Fixed tail: float(30.0) at +5, uint32(17) at +9

  42 of 207 files contain actors (174 total actors found)

[Anim State Records: after actor base, count = animIndicator - 1]
  Type 2 (44 bytes):
    +0   uint32  type           Always 2
    +4   float   frameTime      Trigger frame (e.g. 5.67)
    +8   uint32  unknown        Reserved
    +12  char[32] name          State name (e.g. "anim_idle", "moving", "swing")

  Type 1 (48 bytes):
    +0   uint32  type           Always 1
    +4   float   frameTime      Trigger frame
    +8   uint32  unknown1       Reserved
    +12  uint32  unknown2       Reserved
    +16  char[32] name          Animation name (e.g. "siegeTank_SiegeFire")

  22 total anim states found across all files

[Global Events: after last actor to EOF]
  Sound Cues (string-scan heuristic):
    Patterns: "SF...", "env_...", "terran_...", "@music_...", "*.wav"
    25 files contain sound cue data

  Conversation Triggers:
    Pattern: "cM..." (e.g. "cM121_EscortSCV")
```

## Cross-reference

NCE files contain the same actor/camera/NAD data as their paired `.ncs` file. The NCS version has structured sound cue records with frame times; the NCE stores events with a heuristic layout. For reliable frame-time extraction, prefer the NCS.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NCS](./ncs.md) | [NAK](./nak.md)
