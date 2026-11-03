# Native Timer Triggers

Use the existing `cTimerTrigger` handler on a nonmodeled, noncolliding type-2
NOT helper such as `script_object` or `script_target`. It repeatedly triggers
linked receivers without player interaction.

This uses Ghost's existing runtime. It is not a new timer language, region,
one-shot delay or replacement script VM.

## GtkRadiant workflow

1. Use the updated SDK with the SC Ghost profile. If necessary, click
   **Reload NOT metadata**.
2. Place the actual `script_object` or `script_target` template.
3. Open **N > Selection**, choose **cTimerTrigger** under **Native handler**,
   and click **Initialize / reset**.
4. Set **Timer interval A (first, then alternating seconds)** and
   **Timer interval B (alternating seconds)**. Both default to `5`.
5. Select the timer first, then a supported trigger receiver, and press
   **Ctrl+K**. Repeat for up to eight receivers.
6. Save and export the ordinary MAP using the matching SDK.

Keep the complete generated parameter group. The native body has ten
parameters with kinds `[2,2,3,3,3,3,3,3,3,3]`, no animation frames and no
inline region geometry. Do not initialize this handler on a modeled prop,
actor or `trigger_region`.

| Editor field | MAP key | Meaning |
|---|---|---|
| Timer interval A | `nod_script_param_0` | Initial delay and every other interval thereafter. |
| Timer interval B | `nod_script_param_1` | Delay after the first pulse and every other interval thereafter. |
| Linked target 1..8 | `nod_script_param_2`..`nod_script_param_9` | Receivers triggered on every expiry. |

Unconnected targets use `-2`. Connections store authoring target names or
imported `nsd:OFFSET` identities; do not enter guessed runtime GUIDs.

## Exact timing and signals

For A=`2` and B=`4`, pulse times relative to native scene startup are:

```text
2, 6, 8, 12, 14, 18, ...
```

The timer starts automatically at `BeginScene`. It fires **all connected
targets on every pulse**, in slot order, and then schedules its next expiry.
The two phases do not select separate outputs or send opposite on/off values.
Every output carries trigger ID `0` and zero numeric payload (`p0=0`).

A receiver decides what that signal means. For the verified two-stop door
workflow, it toggles the requested stop. For an objective region, it does not
perform the nonzero-payload arm operation. AI paths need a relay with the
verified start/stop payload; they are not direct timer receivers.

There is **no verified trigger input**, enable switch, repeat count, one-shot
or player-entry start control. A button or `.nods` `trigger NAME;` cannot
start/stop this handler. Use existing relay/action delays when interaction,
rather than automatic periodic behavior, is required.

An output can also drive an existing `.nods` sequence. For example, connect
the timer to `tick` and include this in the configured script:

```text
nodscript 1;
sequence tick { trigger door; }
```

Here `door` is an already placed, named native door. Normal script expansion
resolves timer -> sequence -> door references before compiling the scene.
There is no new timer declaration in NODScript.

## Interval and capacity limits

New or edited intervals must be finite floats from **0.001 to 599.999 seconds**.
Native scheduling truncates to integer milliseconds and rejects a deadline
ten minutes or more ahead. The SDK rejects zero/sub-millisecond intervals,
negative/nonfinite values and unsupported long delays before publication;
failed exports preserve previous outputs.

The native script timer pool has **128 shared slots**, not 128 slots per
helper. New additions cannot bring the combined startup count of timer triggers,
cyclic damage regions, sound loops and the
[gameplay controls' weighted startup timers](native-gameplay-controls.md#validation-and-imported-scenes)
above 128, including imported owners.
This is only a static upper bound:
doors, relays, AI and other scripts can also need timers. Leave headroom.
The compiler cannot guarantee admission for arbitrary original mission logic.

Unchanged source-backed payloads retain exact bytes, including legacy values
outside the new-authoring limits. Editing either interval applies the current
validation; it does not silently clamp or repair the original body.

## Conformance fixture

The mapping workspace contains:

```text
nod\maps\sdk_068_conformance\13_timer_trigger.map
nod\compiled\timer_trigger_runtime
```

Start at `(0, -6, -2)` and watch the two MS_131 NOD/NOC gates. The left timer
uses A=`2`, B=`4` and its first output. The right uses A=`4`, B=`2` and its
eighth output. Left pulses occur at `2,6,8,12` seconds; right pulses at
`4,6,10,12`. Both initially closed gates should request closing again at
seconds `6` and `12`. No button press or region entry is required.

The original timer callbacks, all eight outputs, scheduling conversion,
ten-minute rejection and 128-slot allocation boundary were executed directly.
The compiled fixture also verifies original gate callbacks and an exact
source-backed NIL/NSD/NPD roundtrip. Its gate probe substitutes scheduling,
dispatch and tracker/physics boundaries; rendered motion, collision and final
game timing remain live-game checks.
