# Native Save / Checkpoint Regions

Attach `cRegionSave` to the real `trigger_region` NOT template to request Ghost's
native checkpoint capture on player entry. This uses the existing executable's
save-data path, not a new save format or a replacement scripting runtime.

## GtkRadiant workflow

1. Use matching updated SDK and GtkRadiant builds; **Reload NOT metadata**.
2. Place `trigger_region`, open **N > Selection**, choose **cRegionSave**, and
   click **Initialize / reset**.
3. Set **Checkpoint initially enabled** to `1`, or use `0` and connect an
   existing button/console/relay to enable it later.
4. Normally leave **Objective marker** at `-1`. If deliberately authoring a
   numbered objective-marker route, use a unique ID from `0` through `15`.
5. Adjust **Trigger local minimum / maximum**, keep owner rotation zero,
   save, and export the ordinary MAP.

Default local bounds are `-1 -1 0` to `1 1 2.4`, relative to the entity origin.
Initialization creates two kind-1 parameters, no animation frames, marker `-1`,
and initial enable `1`.

| Field | MAP key | Values |
|---|---|---|
| Objective marker | `nod_script_param_0` | `-1`: do not register; otherwise canonical `0..15`. |
| Checkpoint initially enabled | `nod_script_param_1` | `0`: wait for a trigger; `1`: accept entry immediately. |
| Local bounds | `nod_region_min`, `nod_region_max` | Each axis needs at least `0.001` units of extent. |

`-1` is serialized as the native all-ones 32-bit value. Do not enter
`4294967295`, `01`, or other noncanonical spellings.

## Entry, triggers and one-attempt behavior

- Entry requires a resolved type-7, player-controlled object passing the
  original `IsSavablePlayer` flags test. AI and uncontrolled objects do not save.
  Unlike sound regions, this is **not** the stricter primary-player test:
  a controlled secondary/possessed object is not explicitly excluded.
- The handler sets its consumed latch **before** calling `SavePoint`.
- It then requests native checkpoint capture using the region owner's GUID,
  displays `SYS_SAVE_INGAME`, and attempts to advance the objective marker.
- Re-entry does nothing once consumed. A later trigger does not clear the latch.
- A trigger only enables the region, regardless of trigger ID or numeric data.
  It does not save immediately, disable it, or rearm it.
- If the player was already inside when it was enabled, leave fully and
  re-enter. The trigger callback does not perform an overlap check.
- There are no linked outputs, delays or configurable one-shot settings.
  The consumed latch is included in native script save/restore.

Select an output-producing console/button/relay first, then the checkpoint,
and press **Ctrl+K**. A `.nods` sequence can likewise trigger its target name.
This only enables the native region; no new `.nods` checkpoint command exists.

**The save hint is not proof of persistence.** The handler consumes the attempt,
shows the hint and advances markers even if `SavePoint` rejects the request.
The wrapper requires a current world and a local player; the underlying
`SaveAll` also requires a valid current mission and first local player.
This captures native save data; it is not itself a guarantee of disk/profile
publication. Use a disposable test installation, back up saves, and verify
actual reload behavior separately. There is intentionally no automatic retry
or invented success/failure output.

## Objective markers

At scene start, IDs `0..15` register the region owner's GUID in the chronicle's
shared objective-marker table. ID `0` additionally requests the initial marker.
The marker position comes from the region owner, not the box center.
Duplicate IDs overwrite registrations; the compiler does not resolve a
mission-wide route for you.

After an accepted player entry, the handler attempts to select the marker after
the chronicle's **current** marker, not necessarily this region's own ID.
Gaps/unregistered IDs leave the current marker unchanged. This attempt also
occurs for marker `-1`, and even after a rejected checkpoint request.
Consequently, do not mix these regions casually with an unrelated marker route.
Registration occurs even when the checkpoint initially starts disabled.

These are numeric waypoint-style markers, not the named OBJECTIVE keys used by
`cObjectiveUpdate`.

## Imported scenes and respawn placement

No-edit source-backed roundtrips preserve exact bytes, including older marker
values, nonboolean enable values, and zero- or two-frame records. Typed edits
validate marker and enable settings as above. Imported primary geometry remains
protected.

Fresh regions deliberately have no respawn frames: the native save path falls
back to the current player's transform. Shipped two-frame records are retained,
but authoring new respawn-frame layouts is not exposed by this workflow.
Do not manually change frame counts.

## Conformance fixture

Open `nod\maps\sdk_068_conformance\15_save_regions.map`, or use the separate
`nod\compiled\save_regions_runtime` package and its walkthrough.

Spawn at `(0, -6, -2)`. The cyan left volume at `(-3, 0, -2)` starts enabled
and registers marker `0`. The gold right volume at `(3, 0, -2)` starts disabled
and registers marker `1`. Its floor console at `(3, -4, -2)` enables it.
Try the right volume before pressing the console, leave it fully, press the
console, then re-enter. Each region makes at most one checkpoint attempt per
fresh scene. Restart the level for an independent test; a restored checkpoint
can retain the consumed latch.

Original startup/entry/trigger, player filtering, all 16 marker slots,
one-attempt failure behavior, consumed-latch serialization, native region
readers and exact NIL/NSD/NPD roundtrips are checked automatically.
Persistence/UI and object lookup boundaries are substituted. Actual save/load,
HUD text, marker display and live overlap tracking remain unverified.
