# Native Fog Control

`cFogSet` is an existing Ghost handler with 11 shipped instances. It can run
on a type-3 region such as `trigger_region`, or an unmodeled, noncolliding
type-2 helper such as `script_object`. The SDK supplies its defaults, limits
and compatibility to GtkRadiant; no Java, Codex or replacement runtime is used.

Use **Reload NOT metadata** in the N panel if GtkRadiant was already open.

## Region authoring

1. Place `trigger_region` and set its origin.
2. In **N > Selection**, choose **cFogSet**, then **Initialize / reset**.
3. Adjust its owner-relative minimum/maximum bounds using the supported
   zero-rotation, axis-aligned volume workflow.
4. Choose **Exit behavior**: 1 restores captured fog; 2 keeps the new fog.
5. Set RGB components, start/end distances and transition seconds.
6. Leave **Smoke level after transition** at 0 for a fog-only visual test.

Entry captures the current world fog and applies the region's settings.
The filter is **any native type-7 player**, including secondary/possessed
and uncontrolled players; it is not the primary-controlled-player filter
used by damage or sound regions. AI actor kind 0 is ignored.

Mode 1 restores that region's snapshot on player exit. Mode 2 leaves the
new fog in effect after exit. There is no initial-enable parameter and no
automatic scene-start application for a fresh controller.

## Console/sequence authoring

Place `script_object`, initialize **cFogSet**, and connect a supported
console/button/relay to it with Ctrl+K. An existing `.nods` sequence can also
trigger a named fog controller.

The first trigger captures the current fog and applies the authored settings.
The second restores that snapshot. This is a toggle: **trigger ID and float
data are ignored**, not explicit set/clear commands. Exit behavior is irrelevant
to a helper without a region. The handler has no linked outputs.

The snapshot is the fog present at activation, not necessarily the worldspawn
baseline. If another controller changed the fog first, that is what gets
captured and restored.

## Native fields

The body has zero frames and kinds `[1,1,1,1,1,2,2,2,1]`.

| Parameter | Meaning | New default |
|-----------|---------|-------------|
| 0 | Player-exit behavior: 1 restore, 2 keep | 1 |
| 1 | Fog mode: 0 off, 1 linear | 1 |
| 2 | Red, integer 0..255 | 45 |
| 3 | Green, integer 0..255 | 60 |
| 4 | Blue, integer 0..255 | 90 |
| 5 | Finite nonnegative start distance, meters | 0 |
| 6 | Finite nonnegative end distance, meters | 20 |
| 7 | Transition seconds, 0..599.999 | 1 |
| 8 | Smoke level requested after transition, 0/1 | 0 |

Linear mode requires `start < end` after float32 conversion. Integer edits
must use canonical digits, without leading zeroes. The editor checks scalar
limits; the SDK also checks the relationship between distances.
Transition zero applies immediately. Positive transitions interpolate native
scene color/start/end only when the old and new fog modes match. Changing
between off and linear applies immediately, even with a positive duration.
Use a linear worldspawn baseline when testing visible linear-to-linear fades.

The transition maximum is below the native 600-second timer rejection.
The handler schedules a deferred smoke request after entry or an applying
trigger; it does not schedule an automatic startup timer. Its runtime timers
share the game's 128-slot pool with other handlers.

## Native limitations

- Fog is global, not clipped to the volume. A region chooses when settings
  change; it does not create localized volumetric fog.
- Regions are **not a restoration stack or priority system**. Each captures
  the current fog independently. Out-of-order exits can replace another
  occupied region's fog with an older snapshot.
- Repeated player entries overwrite the same snapshot and toggle the native
  active flag. Prefer disjoint regions and simple single-player walkthroughs.
- Exit restoration does not restore prior smoke or cancel a queued smoke
  timer. A pending smoke callback can still fire after the player leaves or
  a trigger restores the fog.
- A restoring trigger requests smoke level 0, not a saved smoke baseline.
  Smoke requests also participate in native effect-manager frame arbitration.
  Leave smoke at 0 unless deliberately testing its postprocess behavior.
- Mode changes, immediate updates and concurrent controllers share the native
  scene's transition state. Avoid competing changes during a fade.
- The handler's Save/Restore callbacks store its active flag. On BeginScene,
  a restored active flag reapplies authored fog immediately and requests smoke.
  Its captured prior fog is not serialized by those callbacks. Full checkpoint
  restoration is not established by this authoring workflow.

These are original behaviors, not new SDK scripting guarantees.

## Imported scenes and tests

Keep the exported MAP and `.nodsrc` together. Supported scalar edits use the
same bounds as fresh authoring. Changes to linear mode/start/end also validate
their combined float32 distances. Unchanged legacy values remain byte-exact,
including values outside new-authoring limits; editing another independent
field does not silently normalize them. Invalid edits preserve existing outputs.

Visual source:

```text
scghost-mapping\nod\maps\sdk_068_conformance\19_fog_control.map
```

Separate retained package:

```text
scghost-mapping\nod\compiled\fog_control_runtime
```

Nova starts at `(0,-6,-2)`. The cyan left volume applies blue fog and restores
on exit. The floor console toggles purple fog and its captured snapshot.
Test those before the gold right volume, which keeps warm fog after exit.
All transitions are one second, and smoke is off. Restart for a fresh baseline.

Automated checks execute original region/script dispatch, kind-7 filtering,
fog capture/restore, native scene interpolation and effect-manager smoke writes.
Tests include overlapping snapshots, ignored trigger data/IDs, pending smoke
callbacks, native timer limits and the handler's Save/Restore flag.
The renderer/GPU handoff, object lookup, supplied overlap masks and timer/frame
scheduling are fixture boundaries. SDK/editor builds, metadata, handler
switching, Undo/Redo, native inline readers and exact source-backed triples pass.
Live loading, geometric tracking, rendered fades and smoke visuals still need
in-game confirmation.
