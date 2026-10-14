# NAD Animation Data Model

NAD bone tracks define motion as cubic polynomial keyframe curves:

```
  NAD file
  ┌───────────────────────────────────────────────┐
  │ Header: version=4, numBoneTracks, duration    │
  │                                               │
  │ BoneTrack[0]:                                 │
  │   boneNumber=5, type=ROTATION, numKeys=12     │
  │   ┌─────────────────────────────────────────┐ │
  │   │ Key[0]: frame=0.0                       │ │
  │   │   value=(10.5, -3.2, 0.0)  ← base      │ │
  │   │   b=(0.1, 0.0, -0.5)       ← linear    │ │
  │   │   c=(0.0, 0.02, 0.0)       ← quadratic │ │
  │   │   d=(0.0, 0.0, 0.001)      ← cubic     │ │
  │   │                                         │ │
  │   │ Key[1]: frame=5.0                       │ │
  │   │   ...                                   │ │
  │   └─────────────────────────────────────────┘ │
  │                                               │
  │ BoneTrack[1]:                                 │
  │   boneNumber=0, type=POSITION, numKeys=3      │
  │   ...                                         │
  │                                               │
  │ KeyTags: numKeyTags entries                   │
  │   (tagType, frameNum) — animation events      │
  │                                               │
  │ ShapeTracks: numShapeTracks entries            │
  │   (groupNum, shapeNum, weight keyframes)       │
  └───────────────────────────────────────────────┘

  Evaluation at time t within keyframe span:
    v(t) = value + t·b + t²·c + t³·d

  Rotation tracks: euler angles in degrees (pitch, yaw, roll)
  Position tracks: translation offset from rest pose
```

## Bone Hierarchy Evaluation Order

```
  For each bone (parent-first order):
    1. Look up curRotation[i] from NAD rotation track (or zero if no track)
    2. Look up curTranslation[i] from NAD position track (or zero if no track)
    3. Build local rotation matrix from euler PYR (pitch/yaw/roll)
    4. Combine: localMat = rotation × translate(restPos + curTranslation)
    5. If root (parent=0xFF): globalPose = localMat
       Else: globalPose = globalPose[parent] × localMat
    6. Final: boneMat = globalPose × translate(-invRootOffset)
```

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NAD](../formats/nad.md) | [NOD](../formats/nod.md) | [Dependency graph](./dependency-graph.md)
