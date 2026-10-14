# NOD Index Buffer Encoding

NOD mesh groups reference two index regions within the shared index buffer:

```
  Index Buffer (shared across all mesh groups)
  ┌──────────┬───────────┬──────────┬───────────┬─────┐
  │ grp0     │ grp0      │ grp1     │ grp1      │ ... │
  │ strips   │ lists     │ strips   │ lists     │     │
  └──────────┴───────────┴──────────┴───────────┴─────┘

  Per mesh group:
  • startIdx          — offset into index buffer
  • numStripIndices   — triangle strip index count
  • numListIndices    — triangle list index count
  • baseVertex        — added to each index value
```

**Triangle strips** use degenerate triangles (repeated indices) as separators between sub-strips. Conversion to triangle lists requires:
- Walking in groups of 3 with a sliding window
- Alternating winding order on even/odd triangles
- Discarding degenerate triangles (any two indices equal)

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NOD](../formats/nod.md) | [Dependency graph](./dependency-graph.md)
