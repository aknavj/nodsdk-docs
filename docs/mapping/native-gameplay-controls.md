# Native Gameplay Controls

This batch adds 14 verified handler workflows across effects/particles,
movers, object manipulation, messaging and mission transitions.
It uses original Ghost scripts, not Java, Codex or a replacement runtime.
Pickups and unrelated region handlers are unchanged.

Use matching SDK/editor builds and **Reload NOT metadata** in the N panel.
Choose an appropriate NOT placement, select its native handler, then
**Initialize / reset**. SDK metadata supplies defaults, fields, restrictions,
frame setup and connections.

See [Supported trigger capabilities](trigger-capabilities.md) before wiring
controls. An object reference is not necessarily a trigger output.

## Effects and particles

| Handler | Owner | Editable parameters |
|---------|-------|---------------------|
| `cParticleToggle` | Unmodeled, noncolliding effect-bearing type-2 emitter, e.g. `steamJetBig` | 0: initially enabled; 1: auto-disable seconds, 0..599.999 |
| `cEffectAdd` | Unmodeled/noncolliding type-2 helper | 0: effect resource; 1: add on startup / triggers remove; 2: triggered lifetime, 0..599.999 seconds |
| `cObjectExplosion` | Unmodeled/noncolliding type-2 helper | 0: repeat base, 0.001..599.999 seconds; 1: random additional seconds; 2: suppress automatic startup; 3: spawned explosion template |
| `cRain` | Unmodeled/noncolliding type-2 helper | 0..2: finite native rainfall scalars; 3: fixed native mode 0 |

`cParticleToggle` controls its owner's existing emitter, not a referenced target.
Ordinary type-0 triggers toggle it. A positive duration queues a disable
callback; repeated toggles do not cancel older callbacks. When initially
enabled, a positive duration also queues a startup disable. Native type 2
uses data0=1 to disable, otherwise enable; ordinary Ctrl+K/sequence connections
do not expose that alternate trigger type.

`cEffectAdd` parameter 1 is a fixed operating mode, **not a live toggle**.
With mode 0, each trigger requests an effect and optionally queues removal by
resource name. With mode 1, startup adds it and every trigger removes it.
Repeated starts may create duplicate effects/timers. A pending removal can
affect a subsequently added effect of the same name.

`cObjectExplosion` starts suppressed by default. If enabled for automatic use,
each timer requests a spawn and schedules another interval of base plus a
native random fraction of the additional interval. Their binary32 sum must
remain below 600 seconds. A trigger requests one explosion and cancels this
owner's timers; it does not resume automatic operation. The resource is a NOT
object template, not an arbitrary effect filename.

`cRain` applies global rainfall on fresh startup. Triggers request zero rainfall
with native mode 1 and mark this owner disabled. Further triggers do not
reapply its original values. It is not a rain toggle or a restoration stack.
The three floats are deliberately labeled native scalars rather than
unverified wind/density controls.

All four have zero animation frames and no linked trigger outputs.

## More advanced movers

### `cObjectMove`

Attach to a modeled type-2 prop. The SDK publishes tracker MOVETYPE 2 and
PHYSICS_FLAGS 1024, as for the existing door workflow.

Fresh authoring has two translation frames. Frame 0 matches the owner's
origin; both rotations match its native orientation. Stops must be at least
0.001 meters apart, with a finite positive tracker rate.

- Parameter 1: startup delay, 0..598.999 seconds. **Native startup adds one
  second** before scheduling; the default 0.1 therefore starts after 1.1.
- Parameter 2: positive translation speed in meters/second.
- Parameter 3: dwell between arrivals, 0..599.999 seconds.
- Parameter 4: repeat the out-and-back path; default 0.
- Parameter 6: automatic startup; default 0.
- Destination frame 1, interpolation mode, notification references and portal
  settings remain fixed for this verified slice.

An ordinary trigger starts the path while idle. Repeated triggers while busy
are ignored. With repeat off it travels to frame 1, dwells, returns to frame 0
and becomes reusable. Repeat mode continues the native arrival/timer chain.
An ordinary zero-data trigger is **not a stop command**.
Native lockdown/resume uses separate trigger types, not relay data0 alone.

### `cObjectRotate`

Attach to a modeled type-2 prop. The SDK supplies tracker properties and one
pivot frame. Its position initializes at the owner; its nonzero native degree
vector defaults to `0 0 90`. This is not an editor pitch/yaw/roll vector in
radians.

Parameter 1 is positive native rotation speed; 7 selects automatic startup;
8 selects native arrival reversal. Frame selector 0, ramps, delay, interpolation,
notification references and snapshot/reset mode remain fixed.

Type-0 input toggles the active rotation-request chain. Turning it off prevents
subsequent arrival requests; it does **not** forcibly stop a turn already in
progress in this zero-stop-ramp slice. Other ordinary float payloads do not
select angles. Native reversal can issue two initial forward requests before
its first reverse; it is not an authored two-position door toggle.

These are controlled authoring slices, not arbitrary waypoint counts or a
replacement for mission-specific movers.

## Object manipulation

| Handler | Owner and fields | Ordinary trigger |
|---------|------------------|------------------|
| `cObjectSetCollision` | Colliding modeled type-2 prop; parameter 1 chooses initial collision/visibility | data0=1 requests collision on and alpha 1; other values request off and alpha 0, via a scheduled callback |
| `cObjectSetColor` | Modeled type-2 prop; 0 optional shader prefix, 1 startup shader frame, 2 triggered shader frame | Requests parameter-2 shader frame; ignores incoming data/type |
| `cObjectRemove` | Helper; parameters 0..7 are objects to remove | Requests removal of each referenced object; native region targets receive a disable flag instead |
| `cPhysicsActivate` | Modeled colliding type-2 physics prop, MOVETYPE 3, e.g. `crate_25kg` | If parameter 1 is enabled, unpins and requests active physics |

`cObjectSetCollision` fresh authoring fixes manual mode, startup offset and
automatic cycle durations. It changes visibility as well as collisions.
Use a relay/sequence with data `1 0 0 0` for enable; an ordinary zero-data console
requests disable. Native types 2/3/4 have additional inversion/contact-count
behavior outside the ordinary authoring workflow.

Despite its name, `cObjectSetColor` does **not** expose RGB. It schedules shader
frame selection, initially parameter 1 and subsequently parameter 2. A blank
prefix selects all matching shaders. The model/shader must actually provide
the requested frames; syntax checks do not prove a visible material change.

Removal references are **not linked outputs**. Enter their symbolic targetnames
in the fields, rather than drawing outgoing Ctrl+K action connections.
Do not target the player or critical mission actors without understanding
the original runtime consequences.

`cPhysicsActivate` parameter 0 controls startup pinning. The native handler
also enables activation events and re-pins when it receives a deactivation
event. A trigger is not a pin/unpin toggle and does not create a physics body
on a nonphysics template.

## Player messaging

`cHelpText` uses a helper with one deployed localized text key. Ordinary input
shows/hides the native help UI. Showing queues a 120-second timeout. If that
timeout finds the help active, it hides it and consumes this owner's latch;
later triggers then do nothing. Hiding early does not cancel the timeout.
Multiple owners share a global UI object; avoid overlapping help controllers.
This is not arbitrary literal text or an invented duration setting.

`cPopupConversation` supports helpers and native regions. Parameters 0/1 are
optional portrait resources; 2..9 are **one to eight contiguous deployed briefing
keys**. Parameter 10 is the linked completion target. Do not leave gaps:
the original Init counts nonempty keys, then playback indexes that count.

The first trigger, or qualifying primary-player region entry, consumes its
one-shot latch and queues the authored briefing requests. Incoming type/data
do not choose a line. A matching last-key `BriefingRemoved` notification sends a type-0,
zero-data trigger to parameter 10. That notification is not proof every line
played successfully, and repeated matching notifications can repeat the output.

Help keys, briefing keys, portraits, localization and associated audio must be
available in the deployed mission/resources. The SDK does not author chronicles
or transplant mission dependencies.

## Mission transitions

**Use a disposable test installation: these controls can leave or reload a level.**

`cMissionNext` supports helpers and regions. Its one float parameter is read by
the native integer getter: fractional seconds are truncated. Positive whole
seconds schedule Timer10; a result of zero uses 0.1 seconds. Values are
0..599.999. The timer checks the current actor's native guard before requesting
the next mission. Qualifying player/vehicle region entry requests it immediately,
without using the authored delay. There is no one-shot latch or linked output;
repeated triggers may queue multiple requests.

`cLevelChange` uses a native region. Parameter 0 is the deployed level basename;
1 is fixed native arrival selector 0; 2 controls native transition flag 8;
3 selects initially disabled state.

- Every trigger toggles the enabled byte, regardless of type.
- data0 exactly **99** additionally attempts an immediate level-change request
  for the current player, subject to the native actor guard.
- Enabled qualifying player/vehicle entry requests the configured level.
- The native path also requests the loading hint and player revive operation.

data0=0/1 are not explicit disable/enable commands. data0=99 also toggles state;
do not treat it as a side-effect-free command. Destination existence, entry
compatibility, mission state transfer and actual loading remain runtime concerns.

## Validation and imported scenes

Fresh resources/keys use nonempty names of at most 31 ASCII cue-style characters,
without paths, except explicitly optional portrait/shader fields.
These checks validate syntax, not successful resource loading.
Fixed fields reject edits; finite float/integer bounds are shared by ordinary
and edited source-backed authoring. Unchanged imported legacy values and
variable frame bodies remain exact. Keep the matching `.nodsrc`.

The shared 128-slot startup guard now includes these controls' known startup
timers: timed initially enabled emitters, automatic explosions/translations,
shader-frame initialization and collision initialization, alongside timer
triggers, cycling damage and sound loops. Initially hidden collision owners
need two startup slots. Trigger-time timers and other scripts still need
headroom; the guard is not a runtime capacity guarantee.

## Conformance and remaining scope

Five separate ordinary MAPs are in the mapping suite:

| Case | Retained package |
|------|------------------|
| `21_effects_particles.map` | `compiled\effects_particles_runtime` |
| `22_advanced_movers.map` | `compiled\advanced_movers_runtime` |
| `23_object_manipulation.map` | `compiled\object_manipulation_runtime` |
| `24_player_messaging.map` | `compiled\player_messaging_runtime` |
| `25_mission_transitions.map` | `compiled\mission_transitions_runtime` |

Each package has its own walkthrough and hash manifest. Original script
callbacks and translation/rotation wrappers execute under the hash-pinned
native fixture. Resource/effect loading, emitter virtual operations, world rain,
physics/tracking, model shader/alpha, UI/localization, mission handoff and
actor revive are substituted boundaries. Exact source-backed triples pass.
Live rendered effects, physical motion/collision, UI/audio and actual transitions
remain unverified.

Complex `cLightning`, `cConstraint`, `cConstraintBreaker`, specialized
`cDoorSliding` and actor `cFrameMover` workflows are not exposed by this batch.
Their presence in original scenes is not a verified editable contract.
