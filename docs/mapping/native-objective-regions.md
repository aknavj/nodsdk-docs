# Native Objective Regions

Use a native objective region to complete or show an existing mission objective
when the player enters a volume. The region can also trigger a linked door or
another supported receiver.

This workflow attaches the game's `cObjectiveUpdate` handler directly to a
native type-3 region such as `trigger_region`. It is not a `cRegionTrigger`
preset, a new scripting language, or a way to create mission objectives.

## Before you start

- Use matching SDK and GtkRadiant builds that expose `cObjectiveUpdate`.
- Configure the editor's authoring workspace, original asset root and SDK tools.
- Find the objective key in the **active mission's deployed chronicle**.
  Chronicle mission definitions contain `MISSION`, `OBJECTIVE` and `ENDMISSION`
  entries; the extracted Ghost assets use `Misc\ghostsp.nsc`.
- Keep original game assets read-only. Work with workspace copies and deploy
  only to your test installation.

An objective key is an identifier, not the text displayed to the player. For
example, `1_0_0_OBJ_00` is declared in the extracted Test Room mission. Do not
assume that it exists in another mission or in a different deployed chronicle.

> **Important:** The compiler checks key syntax, not membership in your active
> mission. A successful export does not prove that the game can display or
> update that objective. This feature does not create chronicle entries or
> localized objective text.

## Quick start in GtkRadiant

1. Place the actual `trigger_region` NOT template.
2. Select it and open **N > Selection**.
3. Choose **cObjectiveUpdate** under **Native handler**, then click
   **Initialize / reset**.
4. Set **Complete deployed objective (key)** and/or
   **Show deployed objective (key)**. At least one must be nonempty.
5. Leave **Objective region initially enabled** at `1` for a normal
   player-entry objective.
6. Set **Trigger local minimum** and **Trigger local maximum** to enclose the
   desired player path. Keep the region's rotation at zero.
7. Optionally connect the region to a supported trigger receiver as described
   below.
8. Save and export using **Bsp > Convert NOD level...**. Deploy the matching
   outputs and test in the intended mission.

Initializing creates the complete native parameter layout. Do not manually
change its parameter count, parameter kinds or frame count.

## Editable fields

| Editor field | MAP key | Meaning |
|---|---|---|
| Objective region initially enabled | `nod_script_param_0` | `1`: armed at level start; `0`: initially unarmed. Default: `1`. |
| Complete deployed objective (key) | `nod_script_param_1` | Existing objective to mark complete. May be blank when the show key is set. |
| Show deployed objective (key) | `nod_script_param_2` | Existing objective to show. Does not itself complete that objective. May be blank when the complete key is set. |
| On objective update: linked target | `nod_script_param_6` | Optional object reference, normally assigned with **Ctrl+K**. |
| Trigger local minimum | `nod_region_min` | Local lower corner of the volume. Default: `-1 -1 0`. |
| Trigger local maximum | `nod_region_max` | Local upper corner of the volume. Default: `1 1 2.4`. |

Objective keys may contain only ASCII letters, digits and underscores, with a
maximum length of 31 characters. Do not enter a file path, a sentence, spaces
or quotation marks.

For example:

| Complete key | Show key | Requested result |
|---|---|---|
| `1_0_0_OBJ_00` | Empty | Complete the Test Room objective, if declared in the active mission. |
| Empty | An existing mission objective key | Show that objective without completing it. |
| An existing mission objective key | Another existing mission objective key | Complete one objective and show the next. |

### Volume placement

Bounds are local to the entity origin, not absolute world coordinates.
For example:

```text
origin             0 -1 -2
nod_region_min    -1 -1 0
nod_region_max     1  1 2.4
angles             0  0 0
```

This describes a volume from `(-1, -2, -2)` to `(1, 0, 0.4)` in MAP coordinates.
Place it so the player's path passes through its interior.

Use finite values and positive thickness on every axis; the compiler requires
at least `0.001` units of extent per axis. Rotated authored regions are not
currently supported. The editor wireframe follows the authored bounds.

The SDK generates the required native inline region model and sector tail.
You do not need to supply a separate NOD or NOC asset for this volume.

## Connecting a door or another receiver

1. Place and configure the receiving object with a supported native handler.
   For a translating model door, initialize `cDoorMove`, then set its speed
   and closed/open frame positions.
2. Select the objective region **first**, then the receiver.
3. Press **Ctrl+K**.
4. Check **On objective update: linked target** on the region.

This objective workflow exposes one linked output. It sends a native trigger
with `p0=1` after the objective update. It is not an editable delay, arbitrary
message payload or list of multiple outputs.

New object references use `targetname`; imported references can use `nsd:OFFSET`.
Use selection-based connections rather than guessing serialized indices or
runtime GUIDs.

The conformance example uses `MS_131_Door.nod` with matching
`MS_131_Door.NOC` collision. Merely connecting a door does not configure its
model, collision resource, movement frames or clearance.

## Activation and one-shot behavior

For normal player entry:

- The region must be enabled.
- The entering object must be the player; AI entry does not activate it.
- The handler applies the configured complete/show operations, then triggers
  the optional linked target.
- It fires once. Leaving and re-entering does not repeat the update or toggle
  the door again.

Native trigger input is also supported:

| Incoming payload | Native behavior before the region has fired |
|---|---|
| Nonzero `p0` | Arm the region and check whether the current player is inside. Fire immediately if inside; otherwise wait for player entry. |
| `p0=0` | Fire if the region is already enabled. This is not a player-overlap check. |

Once fired, further entry or trigger input is ignored. Do not assume that a
generic trigger connection carries the payload needed to arm an initially
disabled region; check the sending handler's behavior.

The native mission update is not a success-conditioned branch: linking a door
does not make it depend on successful objective-name resolution or visible HUD
feedback. Verify the deployed mission data separately.

## Choosing between region handlers

| Requirement | Handler |
|---|---|
| General entry/exit triggers, actor filters, configurable one-shot state, multiple enter targets | `cRegionTrigger` |
| Complete/show existing mission objectives on player entry, with one verified linked output | `cObjectiveUpdate` |

Do not treat an ordered pair of ordinary trigger checkpoints as a mission-HUD
objective. Conversely, use `cRegionTrigger` when no mission objective needs to
change.

Changing or resetting a handler replaces its parameters. Switching between
these region handlers preserves custom local bounds, but not the previous
handler's objective keys or connections. Clearing the authored handler with
**None (template default)** also removes the authored bounds. These operations
support Undo/Redo.

## Export and deployment

- **Ordinary MAP:** export through the regular MAP converters.
- **Imported/source-backed scene:** retain the matching `.nodsrc` and use the
  source-backed `NODscene` export workflow. Do not replace it with an ordinary
  geometry rebuild.
- Deploy the matching NIL/NSD/NPD outputs and required assets for the level.
  The objective handler is stored in the NSD; copying only the NIL cannot
  update it.
- Keep the intended mission/chronicle active. Renaming a level file does not
  create the corresponding mission objective definitions.

New objective authoring exposes only the fields listed above. Briefings,
savepoints, logbook unlocks and additional callbacks remain read-only at safe
defaults. There is no objective picker, chronicle editor, general-purpose
script editor in this feature. For deployed cutscene playback, use the separate
[native cutscene workflow](native-cutscene-playback.md).

## Conformance map

In the mapping workspace, open:

```text
nod\maps\sdk_068_conformance\11_objective_region.map
```

Its retained package is:

```text
nod\compiled\objective_region_runtime
```

The package provides Test Room filenames, a manifest and sound-bank aliases.
Follow the package manifest and keep matching outputs together.

1. Run the package as `1_0_0_Test_Room` under a mission that declares
   `OBJECTIVE 1_0_0_OBJ_00`.
2. Start at `(0, -5, -2)` and walk along +Y into the blue/gold volume centered
   at `(0, -1, -2)`.
3. Expect the objective completion request and the MS_131 gate to rise once.
4. Leave and re-enter. The gate must not toggle again.

Automated checks execute the original region/sector readers and native
objective/door callbacks. They also cover nonplayer/disabled filtering,
one-shot state, reference resolution and byte-identical no-edit roundtripping.
Objective API and rendering/physics boundaries are substituted in those checks.
Visible mission HUD updates, live player overlap, collision clearance and final
door movement still require an in-game check.

## Troubleshooting

| Symptom | Check |
|---|---|
| `cObjectiveUpdate` is absent from the handler list | Confirm the selected object is a compatible native type-3 NOT region. Use matching updated SDK/editor builds and reload the NOT catalog. |
| Export requires a complete or show key | Fill at least one of the two objective-key fields. |
| An objective key is rejected | Remove spaces, paths and punctuation; use at most 31 ASCII letters, digits or underscores. |
| Export reports a dangling object reference | Confirm the receiver still exists and reconnect it using **Ctrl+K**. |
| Player entry does nothing | Check initial enable state, zero rotation, local bounds, previous activation and that the updated NSD was deployed. |
| The gate moves but the HUD does not change | Verify the objective key belongs to the active deployed mission and has the required display/localization data. Export success and linked output are not proof of HUD success. |
| The gate remains blocked or clips the player | Check its NOD/NOC pair, frame positions, opening dimensions and movement clearance. |

## Related documentation

- [Mapping guides](README.md)
- [Native cutscene playback](native-cutscene-playback.md)
- [Templates](../implementation/templates.md)
- [Level relationships](../implementation/level-relationships.md)
- [NSD scene format](../formats/nsd.md)
- [NOD model format](../formats/nod.md) and [NOC collision](../formats/noc.md)
- [In-game debug tools](../game/debug-tools.md)
