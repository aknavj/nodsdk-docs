# Native Cutscene Playback

Attach `cCutscenePlay` to an existing NOT template to request playback of a
deployed NCS cutscene. Use a `trigger_region` for player-entry activation or a
nonmodeled, noncolliding helper such as `script_object` for activation by a
button, relay or another supported trigger source.

This is a playback workflow for the existing Ghost runtime. It does not create
camera tracks, actors, animations, conversations or new cutscene files.

## Requirements and resource safety

- Use matching SDK and GtkRadiant builds exposing `cCutscenePlay`.
- Choose an existing NCS resource from the deployed `Cutscenes` assets.
- Keep the required models, animations, cast objects, sound banks and mission
  setup available. A filename alone does not transplant a mission cinematic.
- Work in a separate authoring workspace; do not modify original assets.

Enter a basename with or without `.ncs`, for example `0_Test_Cutscene.ncs`.
Names accept 1..31 ASCII letters, digits, underscores or hyphens, plus the
optional `.ncs` suffix. The suffix is case-insensitive. Paths, spaces, `.nce`
references and video filenames are not accepted.

The compiler checks filename syntax, not the resource's existence or all of its
dependencies. NCS files can contain mission-specific behavior and even request
a level change. Restore them in their intended context unless their contents
have been checked. The conformance fixture below uses a verified camera-only
test resource without a map-switch flag.

## Player-entry region

1. Place the actual `trigger_region` NOT template.
2. Select it and open **N > Selection**.
3. Choose **cCutscenePlay** under **Native handler**, then click
   **Initialize / reset**.
4. Set **Deployed cutscene (NCS name)**.
5. Leave **Cutscene one-shot** at `1` for the first test.
6. Adjust **Trigger local minimum** and **Trigger local maximum** so the player
   passes through the volume. Keep the region's rotation at zero.
7. Optionally connect its finish/stop output to a receiver.

Region bounds default to `-1 -1 0` and `1 1 2.4`, relative to the entity origin.
They must be finite, have at least `0.001` units of extent per axis, and use
zero owner rotation. The SDK generates the native inline model and sector tail.

Normal entry is player-only. AI entry does not request playback, and leaving
the region does not stop the cutscene.

## Triggered helper

1. Place a compatible helper such as `script_object` or `script_target`.
2. Initialize **cCutscenePlay** and enter its deployed NCS name.
3. Select a supported output source first, then the helper, and press
   **Ctrl+K**.

The helper has no region bounds or automatic player-overlap behavior. Its
native trigger input requests playback regardless of the cause or payload.
Direct trigger activation of a region behaves the same way: it does not
perform the player-entry filter.

Existing `.nods` sequences can use `trigger NAME;` to activate a named native
cutscene receiver already placed in the MAP. This does not add a cutscene
declaration, a camera language or a replacement game VM to NODScript.

## Editable fields

| Editor field | MAP key | Meaning |
|---|---|---|
| Deployed cutscene (NCS name) | `nod_script_param_0` | Required basename, optionally ending in `.ncs`. |
| Cutscene one-shot | `nod_script_param_1` | `1`: accept one playback attempt; `0`: allow repeat requests. Default: `1`. |
| On cutscene finish or stop: linked target | `nod_script_param_14` | Optional native trigger receiver, normally assigned using **Ctrl+K**. |
| Trigger local minimum / maximum | `nod_region_min`, `nod_region_max` | Region-only local volume bounds; absent from helper schemas. |

Initialization supplies the full 20-parameter native layout with no animation
frames. Do not edit the parameter counts or kind fields.

Automatic startup, preloading, video playback, actor removal, positional actor
gates and numbered timeline-event targets remain protected at safe defaults
for new authoring. Existing source-backed payloads are preserved on no-edit
roundtrips; initializing a new handler is not a way to reconstruct arbitrary
mission-specific settings.

### One-shot means one attempt

The native handler records its one-shot state after requesting playback. It is
not conditional on successful playback or visible camera motion. A failed
request can therefore consume the one-shot.

With one-shot disabled, repeat entry or trigger messages can request another
playback while the previous one is active. There is no new queue or
already-playing guard. Start with one-shot enabled and test repeated activation
only when the resource is safe for it.

## Finish/stop output

Select the cutscene owner first, then a supported receiver, and press
**Ctrl+K**. This writes the single exposed output in parameter 14.

The native handler sends this output for both:

- Normal cutscene finish.
- Explicit cutscene stop.

It sends `p0=0`. There is no success-only branch or configurable payload in
this slice. If the receiver is a door, stopping the cutscene can activate that
door too; do not use this output as proof that the cinematic finished normally.

The output does not fire merely because the player entered or playback was
requested. Timeline-event targets are separate native parameters and are not
exposed by this workflow.

New references use `targetname`; imported references may use `nsd:OFFSET`.
Use **Ctrl+K**, not guessed serialized indices or runtime GUIDs.

## Saving, switching and export

Changing/resetting a handler replaces its parameters and connections. Switching
between supported region handlers preserves custom local bounds; clearing the
authored handler removes them. Handler changes and connections support Undo.

Export ordinary MAPs with the regular converters. For imported scenes, keep the
matching `.nodsrc` and use the source-backed workflow. Deploy matching
NIL/NSD/NPD outputs and the required cutscene assets. The owner and handler are
stored in the NSD; deploying only the NIL cannot add playback.

## Conformance fixture

The mapping workspace provides:

```text
nod\maps\sdk_068_conformance\12_cutscene_region.map
nod\compiled\cutscene_region_runtime
```

Use the package guide, manifest and validation results rather than treating it
as a general mission-cinematic transplant. Start at `(0, -6, -2)`: enter the
left marked volume at `(-3, -1, -2)` or activate the right button at
`(3, -5, -2)`. Keep matching Test Room outputs and deployed resources together.

The fixture uses `0_Test_Cutscene.ncs`, verified by the original NCS reader to
contain one camera row, no actor rows and no map-switch flag. It demonstrates
both direct player-entry playback and a button-triggered helper. Its linked
MS_131 doors provide separate finish/stop indicators.

Automated checks cover native callback dispatch, one-shot filtering, both owner
layouts, finish/stop outputs, file parsing, source-backed resource edits and
byte-identical no-edit roundtrips. Renderer, camera execution, player/physics
and final playback boundaries remain outside those checks. Visible camera
motion, input restoration, collision and final door movement require a live
game test.

## Troubleshooting

| Symptom | Check |
|---|---|
| Handler missing from the list | Use an updated SDK/editor and reload NOT metadata. Select a native type-3 region or a compatible nonmodeled/noncolliding type-2 helper. |
| Filename rejected | Use a basename with an optional `.ncs` suffix, no path or spaces, and at most 31 ASCII characters. |
| Entry does nothing | Check volume placement, enablement of the surrounding gameplay setup, previous one-shot attempts and deployment of the updated NSD. |
| Helper does nothing | Check the sending handler, connection and target name. Helpers do not react to player overlap. |
| No camera movement despite a request | Verify the NCS file and all dependencies; inspect the cutscene's camera rows and mission requirements. One-shot may already have been consumed. |
| A different level loads | Check the NCS map-switch flag and embedded map requirement. Do not transplant that resource blindly. |
| Door activates when playback is stopped | Expected: the exposed output handles finish and stop, not successful completion alone. |
| Repeated or overlapping playback | Keep one-shot enabled or arrange the surrounding native controls to prevent repeated requests. |

## Related documentation

- [Mapping guides](README.md)
- [Native objective regions](native-objective-regions.md)
- [NCS cutscene format](../formats/ncs.md)
- [NCE cutscene events](../formats/nce.md)
- [Templates](../implementation/templates.md)
- [Level relationships](../implementation/level-relationships.md)
