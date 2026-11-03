# Native Sound Loops

`cSoundLoop` is an existing Ghost handler found in 21 shipped placements.
It requests owner-bound looping playback shortly after scene startup and
accepts trigger-driven stop requests. It is **not a reusable start/stop toggle**.
No Java, Codex or replacement runtime is involved.

The SDK supplies its schema to GtkRadiant's existing handler interface.
Use **Reload NOT metadata** in the N panel if the editor was already open.

## Authoring

1. Place an unmodeled, noncolliding type-2 helper such as `script_object`.
2. Position it at the intended sound source.
3. In **N > Selection**, choose **cSoundLoop**, then **Initialize / reset**.
4. Enter **Loop cue** using a cue deployed in the level's sound banks.
5. Optionally connect a console/button/relay to this helper with Ctrl+K.

The cue must be a nonempty name of at most 31 ASCII characters, without
a path. The native wrapper strips the final extension before lookup:
`env_terran_lp_small_room.wav` requests `env_terran_lp_small_room`.
Use an exact deployed cue basename, optionally with its `.wav` suffix;
do not assume arbitrary dotted names remain unchanged.

A helper automatically schedules **one timer after 0.1 seconds**. When that
timer fires, it requests playback if its cached handle is zero. There is no
initial-disable setting and no recurring retry timer.
The owner is passed to the native 3D playback wrapper with flag 1 and no
explicit position override. Actual spatialization, attenuation and looping
depend on the native mixer and cue definition, not new SDK audio settings.

## Trigger behavior: stop, not restart

Every incoming trigger follows the same native branch, regardless of trigger
ID or numeric data:

- Cached handle zero: attempt playback and cache the returned handle.
- Cached handle nonzero: request native mixer stop for that handle.

**The stop branch does not clear the cached handle.** After a successful
start, subsequent triggers keep requesting stop; they cannot restart it.
Sending 0/1 data or AI-style start/stop pulses does not change this behavior.
Restart the scene for another ordinary playback test.

A trigger before the startup timer may start the loop early. The timer then
sees the cached handle and does not create another voice. If the early voice
was already stopped, the timer still does not restart it.

If playback fails and returns zero, the startup timer does not retry by itself.
A later trigger can attempt playback again while the cached handle is zero.
This is original runtime behavior, not a guarantee that a missing bank/cue
will be reported in GtkRadiant.

The EndScene callback requests stop and clears the cached handle. No
handler-specific Save/Restore override establishes persistent stop/play state
across checkpoints. There are no output references, completion outputs,
animation frames or region bounds.

## Fields and limits

The native body has zero frames and kinds `[7,2,2,2]`.

| Parameter | Meaning | New default |
|-----------|---------|-------------|
| 0 | Deployed loop cue | Blank; must be filled before compilation |
| 1 | Legacy float, unread by the native callbacks; read-only | 5 |
| 2 | Legacy float, unread by the native callbacks; read-only | 10 |
| 3 | Legacy float, unread by the native callbacks; read-only | 1 |

The three floats have the same values in all 21 shipped instances. The verified
callbacks do not read them. They are deliberately **not labeled as near range,
far range or volume**. New authoring keeps the shipped defaults rather than
inventing controls that do nothing.

Each loop owner consumes one startup timer slot until its 0.1-second callback.
New loop owners, timer triggers, automatically cycling damage regions and the
[gameplay controls' weighted startup timers](native-gameplay-controls.md#validation-and-imported-scenes) share
the SDK's **128-slot startup guard**, including existing imported owners.
Other runtime scripts also need timers; leave headroom.
The guard does not validate audio voice capacity, stream limits or cue contents.

## Imported scenes

Keep the exported MAP and `.nodsrc` together. Supported edited cue strings
use the same resource-name checks as new authoring. Changed legacy float
slots are rejected as read-only.

Unchanged native payloads roundtrip exactly, including shipped blank cues
and legacy float values outside the fresh defaults. Editing the cue does not
normalize those floats. Invalid edits preserve existing compiled outputs.
Compatibility for new authoring is limited to the supported helper workflow;
unchanged original owners retain their native identities and payloads.

## Listening conformance

Open:

```text
scghost-mapping\nod\maps\sdk_068_conformance\20_sound_loops.map
```

Its separate retained snapshot is:

```text
scghost-mapping\nod\compiled\sound_loops_runtime
```

Nova starts at `(0,-6,-2)`. Cyan and gold markers identify the two sources.
Both request startup playback. Each matching floor console requests stop for
its own loop; repeated use does not restart it. Restart the scene to repeat.
Both normalized cue names are present in the retained bank.

Automated checks execute original Init/BeginScene/Triggered/Timer/EndScene,
cue-name normalization, script playback/stop wrappers and the normal mixer
voice-stop flag write. They verify delayed startup, early triggers, playback
failure/retry, ignored trigger data/IDs, owner isolation, stale handles and
preserved ambient/score handles.

Timer scheduling, object/voice lookup and mixer playback are fixture boundaries.
Audio decoding/output, actual looping and spatialization are not emulated.
SDK builds, editor metadata/handler switching/Undo/Redo, shared startup-budget
checks and exact source-backed NIL/NSD/NPD triples pass.
Live loading, audible loops, attenuation and audible stop timing need in-game
confirmation.
