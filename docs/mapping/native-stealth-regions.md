# Native Location-Stealth Regions

`cRegionStealth` is an existing native handler found in 21 shipped region
instances. GtkRadiant exposes its verified parameter body and trigger behavior
through SDK-owned NOT metadata. It is not a new cloaking system.

**Verified scope:** controlled-player entry writes the native location-stealth
field. The tested `GetVisibilityModifier` query does not read that field and
remains unchanged in the automated fixture. No live AI-detection or HUD effect
has been established. Do not interpret the value as an invisibility percentage,
promise a detection-distance change, or use an apparent visual change as the
sole conformance criterion.

## GtkRadiant workflow

1. Use matching SDK/editor builds and **Reload NOT metadata**.
2. Place the actual `trigger_region` NOT template.
3. Open **N > Selection**, choose **cRegionStealth**, and **Initialize / reset**.
4. Set **Location stealth value (native)** and **Stealth region initially disabled**.
5. Adjust **Trigger local minimum / maximum**, keep owner rotation zero,
   save, and export the ordinary MAP.
6. To toggle the entry gate, select a console/button/relay first, then the
   region, and press **Ctrl+K**.

| Field | MAP key | Meaning |
|---|---|---|
| Location stealth value | `nod_script_param_0` | Finite native float32, default `0.5`; no invented percentage clamp. |
| Initially disabled | `nod_script_param_1` | Canonical `0`: enabled; `1`: disabled. Default `0`. |
| Local bounds | `nod_region_min`, `nod_region_max` | Default `-1 -1 0` to `1 1 2.4`, relative to origin. |

Initialization creates kinds `[2,1]`, zero animation frames and native inline
region geometry. Each bounds axis needs at least `0.001` units of extent.
Shipped values include `0.5`, `1` and `5`, all initially enabled.
The native setter does not clamp; signed finite values are preserved rather
than imposing an unverified `0..1` interpretation.

## Verified behavior

- Entry requires a resolved type-7 object with a player-control pointer.
  AI, unresolved and uncontrolled objects do not receive the write.
  This is not the stricter primary-player filter: controlled secondary/
  possessed player objects can receive it.
- If enabled, entry writes parameter 0 to the player's native location-stealth
  field (`player + 0x534` in the verified executable).
- Every trigger flips the entry gate. Trigger ID and numeric data do not select
  enable/disable: two triggers toggle twice, including two zero-data triggers.
- Toggling while a player is inside does not apply or clear the current value.
  Leave fully and re-enter to observe the new entry gate.
- Exit writes **zero immediately**, even if the region is disabled.
- Exit does not restore a saved value or scan for another occupied stealth
  region. An old region's exit can clear a newer region's value in an overlap.
  This differs from sound-region delayed adjacency handling.
- There are no linked outputs, delay, one-shot or native per-handler
  save/restore overrides. Do not assume toggled state persists through saves.

Use disjoint regions for predictable state transitions. Avoid overlapping these
regions with other systems that write the same native field.

Existing `.nods` sequences can target the region:

```text
nodscript 1;
sequence toggle_stealth {
    trigger right_stealth;
}
```

Invoke the sequence from an ordinary output handler. Each activation toggles;
this is not a start/stop command. No new `.nods` language keyword is introduced.

## Imported scenes

Typed value/disabled fields are exposed with the same finite-float and boolean
validation as fresh authoring. No-edit source-backed roundtrips preserve exact
bytes, including legacy nonboolean flags. The original initializer disables
only when that parameter equals `1`; other legacy values are not normalized.
Imported primary geometry remains protected.

## Conformance fixture

```text
nod\maps\sdk_068_conformance\16_stealth_regions.map
nod\compiled\stealth_regions_runtime
```

Spawn at `(0, -6, -2)`. The cyan left volume at `(-2, 0, -2)` writes `0.5`
and starts enabled. The gold right volume at `(2, 0, -2)` writes `5` but
starts disabled. Its floor console at `(2, -4, -2)` toggles it on each activation.
The volumes overlap at X=`-0.1..0.1`, Y=`-2..2`.

Inspect raw state with an instrumented runtime/debugger. Start outside, try the
disabled right side, use its console, then enter again. Test left-to-right
overlap crossing separately: leaving the left region resets zero even while
the right remains occupied. Return to spawn to leave both volumes fully.
No enemies or artificial HUD indicators are added.

Automated checks execute the original region enter/leave paths, script dispatch,
initializer, toggle callback and location setter. They verify controlled-player
filtering, overlap-bit updates, unclamped values, zero reset, unchanged tested
visibility query, inline model consumption and exact NIL/NSD/NPD roundtrip.
Object lookup, supplied overlap masks and scheduling are boundaries; actual
geometric tracking, model interaction, AI/HUD effects and live gameplay are
not verified.
