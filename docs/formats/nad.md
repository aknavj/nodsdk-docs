# NAD — Nihilistic Animation Data

**Extension:** `.nad`
**Type:** Binary
**Version:** v4

Skeletal animation clip format. Bone tracks store keyframes with cubic polynomial coefficients for smooth interpolation.

## File Layout

```
[Header: 16 bytes]
  uint32 version        Always 4
  uint32 numBoneTracks  Number of bone animation tracks
  uint32 flags          Animation flags
  float  duration       Duration in frames (at 30 fps)

[Bone Tracks: N entries]
  Per-track header (8 bytes):
    uint32 boneNumber   Index into skeleton
    uint16 type         0=rotation (euler°), 1=position
    uint16 numKeys      Keyframe count

  Followed by numKeys × Keyframe (52 bytes each):
    float  frame        Time in frames
    vec3   value        Base value (euler degrees or position)
    vec3   bCoeff       Linear coefficient
    vec3   cCoeff       Quadratic coefficient
    vec3   dCoeff       Cubic coefficient

[Key Tags: variable]
  uint32 numKeyTags
  Per tag (8 bytes):
    uint32 tagType      Event type enum (eRenderKeyTag)
    float  frameNum     Frame at which event fires

[Shape Tracks: variable]
  uint32 numShapeTracks
  Per shape track:
    uint8  groupNum, shapeNum
    uint16 numKeys
    Per key (8 bytes):
      float frame, weight
```

## Keyframe Evaluation

Animation values at time $t$ are evaluated using a cubic polynomial:

$$result = value + t \cdot b + t^2 \cdot c + t^3 \cdot d$$

where $t$ is the normalized time within the keyframe interval.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NOD](./nod.md) | [NMS](./nms.md) | [NNB](./nnb.md) | [NAD Animation Data Model](../implementation/animation.md) | [Model Format Relationships](../implementation/model-relationships.md) | [Coordinate system](../reference/coordinate-system.md)
