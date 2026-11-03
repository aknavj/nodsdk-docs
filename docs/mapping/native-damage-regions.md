# Native Damage Regions

`cRegionDamage` is an existing Ghost handler found in 22 shipped type-3 regions.
It supports primary-player damage, an optional automatic on/off cycle and
trigger toggling. Its other-actor and container modes have different semantics;
do not treat all fields as a generic damage or trigger-output system.

The SDK owns the metadata; GtkRadiant consumes it. No Java, Codex or new
runtime is involved. If the editor was already open when the SDK was updated,
use **Reload NOT metadata** in the N panel.

## Authoring

1. Place the actual NOT template `trigger_region`.
2. Set its origin, then open **N > Selection**.
3. Choose **cRegionDamage** and **Initialize / reset**.
4. Adjust the owner-relative minimum/maximum bounds. Use the supported
   axis-aligned volume workflow, with zero rotation.
5. Set **Damage per tick (native)** and **Damage region initially enabled**.
6. Leave both phase durations at zero for a manually controlled region.
7. Leave the other-actor kill and container options at zero for player damage.

New defaults request 5 native damage units immediately on primary-player
entry and every 0.25 seconds while enabled and occupied. Exit stops the next
pending tick; it does not cancel that timer immediately.
Native difficulty, resistance, AI and script modifiers still apply, so
requested damage is not a guaranteed identical health decrement in every mission.

## Fields

The native body has zero frames and parameter kinds
`[2,1,2,2,1,1,1,1,3,3,3,3]`.

| Parameter | Meaning | New default |
|-----------|---------|-------------|
| 0 | Finite nonnegative native damage per tick | 5 |
| 1 | Initially enabled, canonical 0/1 | 1 |
| 2 | Enabled-phase seconds, 0..599.899 | 0 |
| 3 | Disabled-phase seconds, 0..599.899 | 0 |
| 4 | Kill non-primary actors on entry; ignores enable gate, 0/1 | 0 |
| 5 | Native non-primary kill modifier; read-only | 0 |
| 6 | Native damage type; read-only for this workflow | 0 |
| 7 | Primary entry opens listed containers instead of damage, 0/1 | 0 |
| 8..11 | Four container references, not trigger outputs | -2 |

New regions use the verified damage-type-0 workflow. The native kill modifier
is also kept at zero; no unverified flag meaning is invented. Unchanged imported
values, including shipped damage types 1/3 and legacy integer values, remain
byte-exact. Read-only modifiers cannot be changed through source-backed edits.

## Automatic phases

Cycling starts at scene startup only when **both** phase durations are nonzero.
If either is zero, no automatic cycle is scheduled. The initial-enabled setting
selects the first phase.

The original code adds **0.1 seconds to each authored phase**. For enabled=1,
enabled duration 1 and disabled duration 2, the transitions are approximately:

```text
0.0: enabled
1.1: disabled
3.2: enabled
4.3: disabled
6.4: enabled
```

An enable transition immediately damages the current local player if the
region's primary-player occupancy latch is set, then starts quarter-second
ticks. Manual toggles do not reset the automatic phase timer.

The 599.899-second authoring maximum leaves room for the native 0.1-second
addition below `SetTimer`'s ten-minute rejection threshold. Native float32
addition and scheduler acceptance/rejection were checked at the boundary.
New cyclic owners count alongside timer triggers, sound loops and the
[gameplay controls' weighted startup timers](native-gameplay-controls.md#validation-and-imported-scenes)
against the shared 128-slot
startup budget, including existing owners. Active damage ticks and other
scripts also consume that pool: leave substantial headroom.

## Trigger toggling and an important native limitation

Use Ctrl+K from a supported console/button/relay to this region. Existing
`.nods` sequences may also trigger it.

Native trigger IDs 0, 2 and 3 toggle the enabled state. ID 0 is used by the
supported authoring connections. Other IDs do nothing. Float trigger data
is ignored: sending 0 does **not** disable and sending 1 does **not** enable.
Each accepted trigger simply toggles.

Enabling while occupied immediately damages the current local player.
Disabling does not cancel already queued tick timers. If re-enabled before
an old timer runs, the old chain can continue alongside the new chain,
causing duplicate damage ticks. This original behavior is preserved and tested.

Avoid rapid toggles while occupied. After disabling, leave the region or let
the game advance beyond the pending quarter-second tick before re-enabling.
Do not use AI-style start/stop pulse pairs as an enable/disable API.
There is no handler-specific save/restore override; do not assume manual
toggles or the current phase persist across checkpoint restoration.

## Other actors and containers

The default damage path requires the native primary-player test: a controlled
type-7 actor with no old-player possession reference. Secondary/possessed,
uncontrolled and AI actors do not receive ordinary damage ticks by default.

With parameter 4 enabled, **non-primary actor kinds 0 and 7 receive a native
kill request**, not the configured per-tick damage. This includes secondary
players, not just AI. This branch ignores the enabled state and automatic
phases. Original actor death guards still apply.

The same option opens entering type-8 containers with native force 10.
Other non-actor entries can reach the original damage wrapper, which rejects
non-actors; they must not be interpreted as supported damage targets.

With parameter 7 enabled, primary entry opens all four referenced type-8
containers with native force 999, instead of starting normal player damage.
This branch also ignores the enable gate, then forces enabled state to zero.
It does not set the ordinary primary-player occupancy latch.

Set container references directly using a placed object's `targetname` or
an imported `nsd:OFFSET` identity. Leave unused references at -2.
They are **not action outputs**, so do not configure them with Ctrl+K.
Non-container references do not turn into trigger calls.
Container opening/item drops and AI death presentation still need live testing.

## Imported scenes and conformance

Keep the `.nodsrc` beside its exported MAP. Supported edited floats, booleans
and references use the same bounds as fresh authoring; dangling references
fail without replacing existing outputs. Unchanged legacy payloads and opaque
native region geometry roundtrip exactly.

Open the ordinary, no-sidecar source:

```text
scghost-mapping\nod\maps\sdk_068_conformance\18_damage_regions.map
```

Its separate retained snapshot is:

```text
scghost-mapping\nod\compiled\damage_regions_runtime
```

Nova starts at `(0,-6,-2)`. The cyan left volume is enabled; the gold right
volume starts disabled and is toggled by its floor console. The smaller rear
volume uses enabled/disabled durations 1/2 seconds plus the native offsets.
All three request 2 damage units per tick; other-actor/container modes are off.
Do not stand in a hazard indefinitely. Restart for repeatable tests.

Automated checks execute original region dispatch, primary-player filtering,
damage wrappers, native health subtraction, phase callbacks and death guards
in hash-pinned Unicorn. Object lookup, overlap masks, scheduling, first-player
lookup, AI lookup, presentation, container opening and death-side-effect
services are fixture boundaries. Editor metadata, limits, Undo/Redo, native
inline region readers and exact NIL/NSD/NPD roundtrips pass.
Live loading, geometric tracking, HUD feedback and gameplay remain unverified.
