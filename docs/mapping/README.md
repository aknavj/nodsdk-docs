# Mapping Guides

Practical guides for authoring StarCraft: Ghost maps with GtkRadiant and the
OpenNOD SDK. These workflows use the game's existing native objects and handlers;
they do not add a new scripting runtime.

## Gameplay authoring

- [Supported trigger capabilities](trigger-capabilities.md): inputs, outputs,
  relay payloads, native trigger types and state limitations for all 31 workflows.
- [Native gameplay controls](native-gameplay-controls.md): effects/particles,
  advanced movers, object manipulation, messaging and mission transitions.

- [Native objective regions](native-objective-regions.md): complete or show an
  existing mission objective when the player enters a volume, and connect the
  result to a door or another supported trigger receiver.
- [Native cutscene playback](native-cutscene-playback.md): activate a deployed
  NCS resource from a player-entry region or triggered helper, with a verified
  finish/stop output.
- [Native timer triggers](native-timer-triggers.md): automatic alternating
  intervals and up to eight linked outputs, with native scheduling limits.
- [Native sound regions](native-sound-regions.md): primary-player ambience and
  reverb changes, adjacent-zone transitions and delayed world-default restore.
- [Native save/checkpoint regions](native-save-regions.md): player-entry checkpoint
  attempts, trigger enabling, numeric objective markers and native save limitations.
- [Native location-stealth regions](native-stealth-regions.md): verified state
  writes, trigger toggling and exit reset; no claimed AI/HUD effect.
- [Native Zerg-kill regions](native-kill-zerg-regions.md): entry-based native
  death requests, four actor exemptions and original death guards.
- [Native damage regions](native-damage-regions.md): primary-player damage
  ticks, trigger toggling, automatic phases and distinct actor/container modes.
- [Native fog control](native-fog-control.md): temporary/persistent fog regions,
  triggered snapshot toggling and native scene transitions.
- [Native sound loops](native-sound-loops.md): delayed startup, owner-bound
  playback, trigger stops and the native cached-handle/no-restart limitation.

Use matching SDK and GtkRadiant builds. The SDK supplies the available native
handlers, parameter schemas and template compatibility; the editor consumes
that metadata.

## Related documentation

- [Templates](../implementation/templates.md): NOT inheritance and object identity.
- [Level relationships](../implementation/level-relationships.md): level assets
  and their dependencies.
- [NSD scenes](../formats/nsd.md): native scene records.
- [NOD models](../formats/nod.md) and [NOC collision](../formats/noc.md).
- [Debug tools](../game/debug-tools.md): in-game diagnostic overlays.
