# Cross-Format Dependency Graph

```
                            ┌─────────────────────────────────────────────────────────┐
                            │                   TEXTURE FORMATS                       │
                            │                                                         │
                            │    ┌───────┐     ┌───────┐     ┌───────┐               │
                            │    │  DDS  │     │  TGA  │     │  XPR  │               │
                            │    │.dds   │     │.tga   │     │.xpr   │               │
                            │    └───┬───┘     └───┬───┘     └───┬───┘               │
                            │        │             │             │                    │
                            │        │  raw pixel  │  raw pixel  │ packed archive     │
                            │        │  data       │  data       │ of DDS-like imgs   │
                            │        └─────────────┴─────────────┘                    │
                            └──────────────────────┬──────────────────────────────────┘
                                                   │
                                                   │ referenced by name
                                                   │
                       ┌───────────────────────────┼──────────────────────────┐
                       │                MATERIAL FORMAT                      │
                       │                           │                         │
                       │                    ┌──────┴──────┐                  │
                       │                    │     NSA     │                  │
                       │                    │  .nsa .nas  │                  │
                       │                    │             │                  │
                       │                    │ texture   ──┼──→ DDS name     │
                       │                    │ glow      ──┼──→ DDS name     │
                       │                    │ detail    ──┼──→ DDS name     │
                       │                    │ envmap    ──┼──→ DDS name     │
                       │                    │ shaderType──┼──→ Xbox shader  │
                       │                    └──────┬──────┘                  │
                       └───────────────────────────┼──────────────────────────┘
                                                   │
                                                   │ material names referenced
                                                   │ by geometry formats
                                                   │
          ┌────────────────────────────────────────┼────────────────────────────────┐
          │                                        │                                │
          │                               GEOMETRY FORMATS                          │
          │                                        │                                │
          │     ┌──────────────────────────────────┼──────────────────────┐         │
          │     │              ┌────────────────────┤                     │         │
          │     ▼              ▼                    ▼                     │         │
          │  ┌──────┐    ┌──────────┐         ┌──────────┐               │         │
          │  │ NOD  │    │   NMB    │         │   NIL    │               │         │
          │  │ .nod │    │   .nmb   │         │   .nil   │               │         │
          │  │      │    │          │         │          │               │         │
          │  │model │    │ model    │         │ level    │               │         │
          │  │mesh  │    │ bundle   │         │ sectors  │               │         │
          │  │bones │    │ (N×NOD)  │         │ portals  │               │         │
          │  │mats ─┼─┐  │ mats  ──┼─┐       │ lightmap │               │         │
          │  └──┬───┘ │  └────┬─────┘ │       │ mats  ──┼─── NSA        │         │
          │     │     │       │       │       └──┬──┬───┘               │         │
          │     │     └───────┼───────┴──── NSA  │  │                   │         │
          │     │             │                  │  │                   │         │
          └─────┼─────────────┼──────────────────┼──┼───────────────────┘         │
                │             │                  │  │                              │
   ┌────────────┼─────────────┼──────────────────┼──┼──────────────────────────────┘
   │            │             │                  │  │
   │            │    ANIMATION FORMATS           │  │        SCENE FORMATS
   │            │             │                  │  │
   │            │    ┌────────┴───┐              │  │    ┌──────────────────────┐
   │            │    │    NNB     │              │  │    │                      │
   │            │    │    .nnb    │              │  ├───▶│  NSD  (.nsd)         │
   │            │    │ anim refs  │              │  │    │  entity instances    │
   │            │    └─────┬──────┘              │  │    │  positions/rotations │
   │            │          │                     │  │    └──────┬───────────────┘
   │            │          ▼                     │  │           │
   │            │    ┌──────────┐                │  │           │ entity type name
   │            │    │   NAD    │                │  │           │ → .nod model path
   │            │    │   .nad   │                │  │           ▼
   │  NOD bone ─┼───▶│ tracks   │                │  │    ┌──────────────┐
   │  indices   │    │ keyframe │                │  │    │ NOD (entity) │──── NSA ──── DDS
   │  drive     │    │ cubic    │                │  │    └──────────────┘
   │  tracks    │    └──────────┘                │  │
   │            │                                │  │    ┌──────────────────────┐
   │            │    ┌──────────┐                │  ├───▶│  NPD  (.npd)         │
   │            │    │   NMS    │                │  │    │  placement records   │
   │            │    │   .nms   │                │  │    └──────────────────────┘
   │            │    │ motion   │── NAD paths    │  │
   │            │    │ name map │                │  │    ┌──────────────────────┐
   │            │    └──────────┘                │  └───▶│  NLFX (.nlfx)        │
   │            │                                │       │  effect name list    │
   │            │    ┌──────────┐                │       └──────────┬───────────┘
   │            │    │   NAK    │                │                  │
   │            │    │   .nak   │                │                  ▼
   │            │    │ camera   │                │       ┌──────────────────────┐
   │            │    │ keyframe │                │       │  NFX  (.nfx)         │
   │            │    └──────────┘                │       │  effect controllers  │
   │            │                                │       │  particle systems    │
   │            │  COLLISION FORMAT              │       └──────────────────────┘
   │            │    ┌──────────┐                │
   │            │    │   NOC    │                │
   │            │    │   .noc   │                │
   │            │    │ col mesh │                │
   │            │    └──────────┘                │
   │            │                                │
   └────────────┴────────────────────────────────┘

     TEMPLATE FORMAT                      CUTSCENE FORMATS

     ┌──────────┐                         ┌──────────┐
     │   NOT    │                         │   NCE    │
     │   .not   │                         │   .nce   │
     │ entity   │                         │ cutscene │
     │ template │──── defines schemas     │ events   │
     │ hierarch │     for NSD entity      └────┬─────┘
     └──────────┘     types                    │ paired 1:1
                                               ▼
     LOCALIZATION                         ┌──────────┐
                                          │   NCS    │
     ┌──────────┐    ┌──────────┐         │   .ncs   │
     │   NLT    │    │ NLS/NLU  │         │ cutscene │
     │   .nlt   │    │.nls .nlu │         │ camera   │
     │ raw text │    │ key=val  │         │ shots    │
     └──────────┘    │ strings  │         └──────────┘
                     └──────────┘

     UI FORMAT                            TERRAIN FORMAT

     ┌──────────┐                         ┌──────────┐
     │   NUI    │                         │ NRT/NUT  │
     │   .nui   │                         │.nrt .nut │
     │ screen   │──── DDS (icon refs)     │ terrain  │
     │ layouts  │                         │ patches  │
     └──────────┘                         └──────────┘
```

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [DDS](../formats/dds.md) | [TGA](../formats/tga.md) | [XPR](../formats/xpr.md) | [NAD](../formats/nad.md) | [NOD](../formats/nod.md) | [NMS](../formats/nms.md) | [NRT/NUT](../formats/nrt-nut.md) | [NIL](../formats/nil.md) | [NLFX](../formats/nlfx.md) | [NPD](../formats/npd.md) | [NSA](../formats/nsa.md) | [NSD](../formats/nsd.md) | [NLT](../formats/nlt.md) | [NOC](../formats/noc.md) | [NLS/NLU](../formats/nls-nlu.md) | [NOT](../formats/not.md) | [NFX](../formats/nfx.md) | [NUI](../formats/nui.md) | [NAK](../formats/nak.md) | [NCE](../formats/nce.md) | [NCS](../formats/ncs.md) | [NMB](../formats/nmb.md) | [NNB](../formats/nnb.md)
