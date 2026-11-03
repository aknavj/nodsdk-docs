# Native Zerg-Kill Regions

`cRegionKillZerg` is an existing native entry handler found in three shipped
type-3 regions. It requests native actor death when an entering actor has the
Zerg actor-type bit and is not one of four exempt objects.

This uses the original Ghost handler, not Java, Codex or a new runtime.
Use matching SDK and GtkRadiant builds; the SDK supplies the schema.
If the editor was already open when the SDK was updated, use
**Reload NOT metadata** in the N panel before selecting the handler.

## Authoring

1. Place the actual NOT template `trigger_region`.
2. Set its origin before configuring the volume.
3. Open **N > Selection**, select **cRegionKillZerg**, and **Initialize / reset**.
4. Adjust the owner-relative region minimum/maximum. Keep volume rotation zero
   for the supported axis-aligned authoring workflow.
5. To protect a particular placed actor, give it a unique `targetname` and
   enter that name in an **Exempt actor** field.

There are four object-reference parameters, kinds `[3,3,3,3]`, and no frames.
New exemption slots default to `-2` (no object). The original shipped regions
use `-1`; unchanged imported sentinel values remain byte-exact.
Use `targetname` for new objects or the exported `nsd:OFFSET` identity for
imported objects. Do not guess numeric runtime GUIDs.

**These are exemptions, not linked trigger outputs.** Do not use Ctrl+K to
configure them. Set the parameter fields directly. A named spawn helper is not
the identity of the actors it creates and therefore does not exempt its children.

## Verified behavior

- Entry first compares the entering GUID against all four exemption slots.
  Any match suppresses this region's kill request.
- Only native actor kinds 0 and 7 qualify. Their actor-type flags must include
  bit `0x8`; combined flags such as `0x9` still qualify.
- Normal Terran Marine/Nova actors without that bit are unaffected. This is
  not restricted to AI: a type-7 player with the Zerg bit also qualifies.
- Qualifying entry calls the original `ScriptActor::KillActor`, which enters
  `tActor::Kill`. Original death guards still apply. The verified fixture
  sets the native dead flag and health to zero; guarded actors reject death.
- There is no one-shot latch. Another qualifying actor can die on a later
  entry. Re-entering an already-dead actor does not repeat the death callback.
- An exemption belongs to its region only. It does not protect the same actor
  from a different region.

The handler overrides only **Entered**. It has no trigger input, enable/disable
parameter, timer, exit action or linked output. Sending trigger data cannot
toggle it, and `.nods` sequences cannot use it as a trigger receiver.
Do not treat this as a general damage region or a player kill volume.

The handler does not itself scan actors already inside. It responds to native
entry dispatch. Initial overlaps, geometric tracking and model/AI death
presentation must be checked in the live game.

## Imported scenes

Source-backed exports expose the four typed exemption fields. Supported
reference edits are validated before publication; dangling references fail
without replacing the existing output triple. No-edit source-backed
NIL/NSD/NPD roundtrips preserve the original bytes, including opaque region
geometry and unchanged legacy references.

Keep the `.nodsrc` beside its exported MAP. Do not replace an imported native
volume with newly generated brush geometry simply to edit an exemption.

## Conformance map

Open the ordinary, no-sidecar source:

```text
scghost-mapping\nod\maps\sdk_068_conformance\17_kill_zerg_regions.map
```

Its separate retained snapshot is:

```text
scghost-mapping\nod\compiled\kill_zerg_regions_runtime
```

The cyan left region contains an unexempted Zergling and a Marine control.
The gold right region exempts its named Zergling. Nova starts at `(0,-6,-2)`.
The Zerglings have no template AI resource; the Marine's inherited AI resource
is cleared to isolate region handling from combat/navigation. On native entry,
the left Zergling should
receive a death request, the right Zergling should survive its own region,
and Marine/Nova should be unaffected.

Start from a fresh scene for repeatable death tests. If an initial overlap
does not emit entry, arrange a real boundary crossing or use instrumentation;
do not infer a startup scan or invent a trigger-to-kill action.
The package walkthrough describes additional region-local exemption checks.

## Validation boundary

Automated checks execute the hash-pinned original region callbacks,
`tRegion::Enter`/`Leave`, script dispatch, actor-type getter, kill wrapper and
native actor death logic in Unicorn. All four exemption slots, actor masks,
death guards and state writes are checked.

Object lookup, supplied overlap masks, AI lookup, threat direction, death
presentation, corpse awareness/expiration and player-death notification are
fixture boundaries. Native inline region readers, compiler serialization,
editor metadata, handler switching, Undo/Redo and exact source-backed
roundtrips are also checked. Live loading, initial-overlap tracking,
death animations and gameplay remain unverified.
