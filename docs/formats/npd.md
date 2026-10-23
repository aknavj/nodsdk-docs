# NPD — Navigation Graph

**Extension:** `.npd`
**Type:** Binary

AI navigation waypoint graph with **world-space XYZ positions** and directed
links. The original loader copies the position floats unchanged and passes them
to the world's sector lookup. There is no separate approximately 4× XY scale
or compressed/floor-index Z coordinate system.

NPD is not entity placement data; [NSD](./nsd.md) supplies entity placements.
A waypoint ID is a unique navigation identifier, not an NSD entity reference.

## Header (8 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | version | Native reader accepts 1 or 2; fresh writer uses 2 |
| 0x02 | 2 | nodeCount | Number of valid waypoint records |
| 0x04 | 2 | linkCount | Number of directed links |
| 0x06 | 2 | nextUniqueId | Next editor waypoint ID |

## Waypoint Records (nodeCount × 26 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | waypointId | Unique waypoint identifier (u16) |
| 0x02 | 12 | position | World-space XYZ, three float32 values |
| 0x0E | 2 | flags | Node flags; ordinary generated nodes use 0 |
| 0x10 | 1 | component | Connectivity group; native search compares groups first |
| 0x11 | 2 | associationA | Associated-link index; `0xFFFF` when absent |
| 0x13 | 2 | associationB | Associated-link index; `0xFFFF` when absent |
| 0x15 | 2 | sector | Sector index; recomputed by the native loader |
| 0x17 | 2 | firstOutgoing | Array index of the first outgoing link |
| 0x19 | 1 | outgoingCount | Number of outgoing links, at most 255 |

## Directed Links (linkCount × 6 bytes)

Links follow the nodes and are grouped by source node to match each node's
outgoing range. Each entry occupies six bytes:

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0x00 | 2 | fromNode | Source node index (0-based record index) |
| 0x02 | 2 | toNode | Destination node index (0-based record index) |
| 0x04 | 2 | flags | Link flags, not a numeric cost; ordinary generated links use 0 |

Endpoints are **0-based record indices**, not waypoint IDs. The loader derives
normalized direction vectors from endpoint positions. Reverse travel needs a
separate reverse link.

## Bounded Generation Profile

The reconstructed writer supports ordinary flags, absent associations, sector 0,
and weakly connected component IDs 0..254. Special traversal and association
semantics remain unresolved. This profile allows IDs 0..65534, at most 65,535
nodes, 65,535 links, 255 outgoing links per node, and 255 components.
Zero, underflowing, or overflowing squared link lengths must be rejected before
native normalization.

## Legacy SDK Field Names

The version-2 parser's `color` and `extra` fields are byte-preserving packed views
of offsets `0x10..0x19`, not visualization data. `LinkRecord::cost` is a legacy
name for link flags.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NIL](./nil.md) | [NSD](./nsd.md) | [Level Format Relationships](../implementation/level-relationships.md) | [Coordinate system](../reference/coordinate-system.md)
