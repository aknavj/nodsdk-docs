# Native Sound Regions

Attach `cRegionSound` to the existing `trigger_region` NOT template to change
ambience and/or reverb when the primary player enters a volume. It uses
Ghost's existing sound-region callbacks, not positional sound emitters,
new audio assets or a replacement scripting runtime.

## GtkRadiant workflow

1. Use the matching updated SDK and GtkRadiant build. Click **Reload NOT
   metadata** if the new handler is not listed.
2. Place the actual `trigger_region` template and open **N > Selection**.
3. Choose **cRegionSound** under **Native handler**, then click
   **Initialize / reset**.
4. Set **Region reverb preset (-1: unchanged)** and/or
   **Region ambient cue (blank: unchanged)**.
5. Adjust **Trigger local minimum / maximum** around the intended area.
   Keep owner rotation zero.
6. Configure a nonempty world **Ambient loop cue** if leaving the region
   should resume a baseline ambience.
7. Save and export the ordinary MAP using the updated SDK.

Default local bounds are `-1 -1 0` to `1 1 2.4`, relative to the entity origin.
Each axis must have at least `0.001` units of extent. The SDK supplies the full
native inline model and sector tail; do not replace it with a point helper.

| Editor field | MAP key | Meaning |
|---|---|---|
| Region reverb preset | `nod_script_param_0` | `-1`: leave current reverb unchanged; `0..25`: request that native preset. |
| Region ambient cue | `nod_script_param_1` | Blank: leave current ambience unchanged; otherwise request a cue in the loaded level bank. |
| Trigger local minimum / maximum | `nod_region_min`, `nod_region_max` | Local region bounds. |

Initialization creates two parameters, kinds `[1,7]`, and no animation frames.
Both sound settings default to unchanged. Reverb `-1` is stored as the native
32-bit all-ones value. Use canonical decimal IDs; do not assume preset `0`
means "off" or assign unverified acoustic names to the IDs.

Cue names allow at most 31 ASCII letters, digits, underscores, hyphens or dots,
without paths or leading/trailing dots. Legacy names with a suffix such as
`.wav` are accepted. This is a bank cue reference, not a standalone WAV loader.
The compiler checks syntax, not whether the active level bank contains the cue.
Keep matching XSB/XWB files deployed.

## Verified entry and exit behavior

- Entry applies only to the **primary player**, not AI, an unresolved actor,
  or a possessed/secondary player object.
- A nonnegative reverb value is applied on entry. `-1` makes no reverb request.
- A nonempty ambient cue is played unless its stored name exactly matches
  the mixer's current ambient name. Name comparison is case-sensitive;
  differently spelled/suffixed references can request a restart.
- Player exit schedules a cleanup check **0.5 seconds later**.
- If the current player still overlaps any `cRegionSound` in its sector,
  including a re-entered or adjacent sound region, cleanup leaves settings alone.
- Otherwise cleanup restores the **world defaults**, and only for the settings
  that this region specified. It does not restore a saved previous region.

There is no region-settings stack or guaranteed overlap priority. On a shared
seam, the most recent entry can set the active values; delayed cleanup avoids
overwriting an occupied sound region. Prefer adjacent areas with explicit
ambience and reverb settings rather than stacking partial overrides.

**A blank world ambient cue does not stop the old loop.** The original mixer
ignores an empty ambient playback request. Set a real baseline cue when
authoring a region whose exit must return to known ambience.

The handler has no verified trigger input, linked output, enable switch or
one-shot setting. Do not connect a button/relay or `.nods` `trigger` statement
to it. Use `cSoundStinger` for a trigger-activated cue.

## Imported scenes and limits

Supported typed settings are exposed on imported sound regions. No-edit
roundtrips preserve exact bytes; `-1` is displayed consistently instead of an
unsigned number. Unknown legacy values remain unchanged unless edited.
Edits use the same cue/reverb validation as fresh authoring. Imported primary
geometry remains protected; new region bounds do not rebuild an opaque
original model.

Exit checks use the shared native script timer pool. Leave headroom for
other timers, relays and AI. Mixer, tracker and final level playback remain
the original runtime's responsibility.

## Conformance fixture

```text
nod\maps\sdk_068_conformance\14_sound_regions.map
nod\compiled\sound_regions_runtime
```

Start at `(0, -5, -2)`. Walk into the cyan left zone at `(-2, 0, -2)`,
cross the narrow center seam into the gold right zone at `(2, 0, -2)`,
then leave both zones by walking to `(0, 4, -2)` or back to the spawn point.
Wait more than 0.5 seconds outside.

| Location | Ambient cue | Reverb ID |
|---|---|---|
| Baseline/outside | `env_terran_lp_hall2` | `0` |
| Left zone | `env_terran_lp_small_room` | `23` |
| Right zone | `env_terran_lp_hangar` | `2` |

The zones overlap by 0.2 meters at X=`-0.1..0.1`. Cross fully into one zone
rather than standing in the overlap when comparing results.

Automated checks execute original callbacks, primary-player filtering,
the native sector overlap scan, delayed baseline restoration and region model
readers. The retained bank contains all three cue names, and source-backed
NIL/NSD/NPD roundtrip is exact. Mixer, scheduler and tracker boundaries are
substituted; audible transitions, effects, fades and final in-game overlap
tracking still need live conformance testing.
