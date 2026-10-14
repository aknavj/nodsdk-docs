# NMS — Nihilistic Motion Set

**Extension:** `.nms`
**Type:** Text (tab-delimited)

Maps motion state names to animation (.nad) file paths. Used by the animation system to resolve gameplay states to specific clips.

## Format

```
; Semicolon-prefixed comment header
motionName<TAB>animFile.nad<TAB>[flag1]<TAB>[flag2]...
```

Each line maps a motion name (e.g., `idle`, `run`, `attack_01`) to a `.nad` animation file, optionally with flags that control playback behavior.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NAD](./nad.md) | [Naming Conventions That Bind Formats](../implementation/naming-conventions.md) | [NAD Animation Data Model](../implementation/animation.md)
