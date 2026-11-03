# Supported Trigger Capabilities

The SDK currently publishes **31 native handler workflows**. This reference
describes the supported authoring surface, not every callback in the executable.
Existing imported objects can have additional unchanged native behavior.

## Wiring and payloads

- **Ctrl+K:** select a supported output owner first, then its receiver.
- Console/button activation is different from receiving a script trigger.
- Relay actions expose delay and four-float data. Ordinary connections use
  native trigger **type 0**. Changing data does not select another native type.
- In the tables, `data0` means the first float, not an object ID or trigger type.
- `.nods` sequences can target named supported receivers already in the MAP.
  Example: `nodscript 1; sequence start { trigger receiver; }`.
  Use the documented relay data fields for explicit start/stop payloads.
- A native callback or completion notification is not proof that its renderer,
  resource, audio or mission operation succeeded.

NOT-native action graphs reject dangling/incompatible targets and cycles.
Relays retain at most 12 actions; new graphs allow at most 64 transitive actions.
AI paths require verified relay start/stop payloads instead of direct console
connections. Object-removal, exemption and container references are not outputs.

## Existing workflows

| Handler | Supported incoming action | Authored outgoing triggers / limits |
|---------|---------------------------|------------------------------------|
| `cTriggerOnAction` | Player activation, not an incoming script connection | Four targets with delays |
| `cButton` | Player activation, not an incoming script connection | Eight targets; one-shot/lock configuration |
| `cSoundStinger` | Requests cue playback | None; does not replace world ambient/score control |
| `cSoundLoop` | Starts if cached handle zero; otherwise requests stop | None; stopping retains handle, so no trigger restart |
| `cAIFightPath` | Relay pulse: data `1 0 0 0` starts, `0 0 0 0` stops | No exposed outputs; lifetime spawn limit 1..25 |
| `cTriggerRelay` | Schedules its authored actions | Up to 12 target/delay/data actions |
| `cDoorMove` | Ordinary trigger for the verified two-stop door/lift workflow | No exposed notification outputs |
| `cRegionTrigger` | Ordinary input toggles the region enable state | Up to eight configured entry/exit targets |
| `cObjectiveUpdate` | Nonzero data0 arms/checks current player; zero fires only while enabled | One exposed target, data0=1; deployed objective keys required |
| `cCutscenePlay` | Requests playback; input type/data do not select a timeline event | One finish **or stop** target, data0=0 |
| `cTimerTrigger` | None; starts automatically | Eight targets on each alternating expiry, data0=0 |
| `cRegionSound` | None; primary-player region occupancy | None; delayed world-default restore, not a nesting stack |
| `cRegionSave` | Enables checkpoint attempts | None; qualifying entry consumes an attempt, not a guaranteed disk save |
| `cRegionStealth` | Toggles its enabled state | None; entry/exit write native location-stealth state |
| `cRegionKillZerg` | None; native actor entry filtering | None; four references are exemptions |
| `cRegionDamage` | Native types 0/2/3 toggle; data ignored | None; container references are not outputs |
| `cFogSet` | Toggles apply/restore snapshot; input type/data ignored | None; regions also apply/restore on qualifying entry/exit |

See the individual [mapping guides](README.md) for precise state, resource and
save/restore limitations. A timer without an input cannot be stopped by simply
drawing a console connection to it.

## New five-area workflows

| Handler | Supported incoming action | Outputs and important native limits |
|---------|---------------------------|-------------------------------------|
| `cParticleToggle` | Ordinary type 0 toggles the owner's emitter; data ignored | None; positive duration queues disable, without cancelling older timers |
| `cEffectAdd` | Mode 0 adds and optionally queues removal; mode 1 removes the startup effect | None; operating mode is not toggled by input |
| `cObjectExplosion` | Requests one spawn and cancels this owner's timers | None; does not restart automatic explosions |
| `cRain` | Requests global rainfall off | None; repeated input does not restart rain |
| `cObjectMove` | Starts an idle out-and-back path | No exposed notification outputs; busy input ignored; repeat is an authored setting |
| `cObjectRotate` | Type 0 toggles further rotation requests | No exposed notification outputs; stopping does not interrupt the current turn |
| `cObjectSetCollision` | Type 0: data0=1 on/visible; other values off/hidden | None; change is scheduled, not an immediate toggle |
| `cObjectSetColor` | Schedules the authored triggered shader frame | None; not RGB, incoming type/data ignored |
| `cObjectRemove` | Requests removal of up to eight referenced objects | None; region references receive native disable flags |
| `cPhysicsActivate` | Unpins/activates when its authored trigger option is enabled | None; deactivation events can re-pin |
| `cHelpText` | Shows/hides localized help until timeout consumes the owner | None; shared global UI, 120-second timeout |
| `cPopupConversation` | One-shot briefing requests; type/data ignored | One last-key removal target, type 0 and zero data; repeated notifications can repeat output |
| `cMissionNext` | Schedules next-mission request using truncated whole-second delay | None; region entry is immediate; no one-shot guard |
| `cLevelChange` | Every input toggles enable; data0=99 also attempts immediate transition | None; enabled region entry can transition; destination must be deployed |

See [Native gameplay controls](native-gameplay-controls.md) for fields, compatible
owners, conformance rooms and detailed caveats.

## Native types not selected by ordinary connections

The verified original callbacks also show these distinctions. They are **not**
new `.nods` syntax or a promise of an arbitrary-type editor sender:

- `cParticleToggle`: types 0/1/3 toggle; type 2 uses data0=1 for disable,
  otherwise enable. Other types are ignored.
- `cObjectMove`: type 2 uses data0=1 for lockdown and 0 for resume.
- `cObjectRotate`: type 1 is ignored; type 2 has a native lockdown branch.
- `cObjectSetCollision`: type 2 reverses the ordinary data0=1 branch;
  types 3/4 maintain a native contact counter and delayed re-enable.
- `cRegionDamage`: its input filter is distinct from ordinary data values.

Do not substitute relay floats for those native type selectors.
Only explicitly exposed output layouts participate in editor connection
validation; a legacy reference field does not become an event connection.

## Verification boundary

SDK builds, field validation, source-backed edit preservation, generic editor
metadata/handler switching/Undo/Redo, original callback regressions and conformance
roundtrips pass. Native fixtures substitute scheduling, lookup, resources,
presentation and selected runtime services. They do not prove live rendering,
audio, physical tracking, UI localization, checkpoint persistence or level loading.
