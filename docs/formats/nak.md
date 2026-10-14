# NAK — Nihilistic Animation Keyframes

**Extension:** `.nak`
**Type:** Text

Camera animation keyframes paired with cutscene `.nad` files (3 files total).

## Format

```
<keyframeCount>
<time> <fov> <nearClip> <farClip>
<time> <fov> <nearClip> <farClip>
...
```

First line is the keyframe count, followed by one keyframe per line with four float values: timestamp, field of view, near clip plane, and far clip plane distances.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NCE](./nce.md) | [NCS](./ncs.md) | [Coordinate system](../reference/coordinate-system.md)
