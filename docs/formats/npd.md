# NPD — Nihilistic Patrol Data

**Extension:** `.npd`
**Type:** Binary

AI navigation/patrol waypoint graph for a level. Contains waypoint nodes with
positions in navigation-map coordinate space (NOT world-space) and an adjacency
list defining connectivity between waypoints for AI pathfinding.

**Important:** NPD is NOT entity placement data. Entity world-space positions
are defined in the NSD file. NPD positions are in a separate 2D navigation-map
coordinate space with a compressed Z axis (floor/elevation index). The
`waypointId` field is a unique waypoint identifier, not an NSD entity reference.

## Header (8 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | version | Format version (= 2) |
| 0x02 | 2 | nodeCount | Number of valid waypoint records |
| 0x04 | 4 | unknown | Reserved / unknown |

## Waypoint Records (nodeCount × 26 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | waypointId | Unique waypoint identifier (u16) |
| 0x02 | 12 | position | Navigation-map position (3 floats, XYZ) |
| 0x0E | 2 | flags | Waypoint flags (0=normal, 1=special, 1024=?) |
| 0x10 | 4 | color | Debug visualization color (ARGB) |
| 0x14 | 6 | extra | Additional data (3 × u16) |

## Adjacency List (remaining bytes after records)

After the waypoint records, the rest of the file is an adjacency list defining
connections between waypoints. Each entry is 6 bytes:

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | fromNode | Source node index (0-based record index) |
| 0x02 | 2 | toNode | Destination node index (0-based record index) |
| 0x04 | 2 | cost | Edge cost/weight (0, 1, 2, 3, or special values) |

The adjacency list uses **0-based record indices** (not waypointIds). Typical
levels have ~3–4 links per node, forming 1–3 connected components.

## Coordinate Space

NPD positions use a navigation-map coordinate system distinct from NSD
world-space:
- X/Y axes cover approximately the same spatial extent as NSD but at ~4× scale
- Z axis is compressed to floor/elevation levels (e.g., -10.9, -7.6, 1.2, 2.9, 9.9)
- Not directly usable for entity rendering; only for AI pathfinding logic

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NIL](./nil.md) | [NSD](./nsd.md) | [Level Format Relationships](../implementation/level-relationships.md) | [Coordinate system](../reference/coordinate-system.md)
