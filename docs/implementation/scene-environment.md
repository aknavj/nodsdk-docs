# NSD Scene Environment

`MAPtoNSD` reads scene-wide settings from the MAP's `worldspawn` entity and
writes them to the NSD. These control sky references, audio references, fog,
camera draw distance, and reverb. They do not change NIL materials, lighting,
or collision.

The [NSD reference](../formats/nsd.md#bounded-point-scene-output) describes
the binary fields. The sections below describe the supported authoring settings.

## Sky References

| Key | Value |
|-----|-------|
| `nod_sky_model` | Sky model filename ending in `.nod` |
| `nod_sky_animation` | Sky animation filename ending in `.nad` |

Use filenames without directory paths, at most 31 ASCII characters long.
Allowed characters are letters, digits, `_`, `-`, and `.`; extension checks
ignore case. Empty or omitted values leave the reference unset.

The two references are independent: setting an animation does not select a
model. The compiler stores the names but does not check that the assets exist
or verify how the sky renders.

## Music and Ambient References

| Key | Value |
|-----|-------|
| `nod_music` | Music reference |
| `nod_ambient_loop` | Ambient-loop reference |

Names may contain up to 31 ASCII characters: letters, digits, `_`, `-`, and
`.`. Paths and leading or trailing dots are not allowed. Extensions are
optional; the native loader removes the last extension. Empty or omitted
values leave the reference unset.

The game passes the ambient reference to its ambient-playback routine at
level startup. Audible playback and playback from the music field have not
been verified. These settings do not override the game's score-selection logic.

## Fog

| Key | Value |
|-----|-------|
| `nod_fog_mode` | `none` (default) or `linear` |
| `nod_fog_color` | Three finite RGB numbers, each in `0..255` |
| `nod_fog_start` | Distance where linear fog starts |
| `nod_fog_end` | Distance where linear fog ends |

Linear fog requires all three color/distance settings. Distances use MAP
world units and must satisfy `0 <= start < end` after conversion to the
stored 32-bit floats. Color and distance settings are rejected unless the
mode is `linear`.

Fog does not set the camera's draw distance; configure far clipping separately.

## Far Clipping

`nod_far_clip` sets the camera's far clipping distance in world units. Use a
finite positive number, or `0` to keep the runtime default. Omitting the key
also keeps the default: 300 units in the examined debug build.

Choose a distance beyond the camera's near plane, which this setting does
not change. Fog and physics bounds do not determine this value. Empty,
negative, nonfinite, or unrepresentable values are rejected, including
positive values too small to remain nonzero as a 32-bit float.

## Reverb

`nod_reverb` selects a native preset by number, from `0` through `25`.
Omitting it uses preset `0`; this is not a documented "off" setting.
Use plain decimal integers without signs, leading zeros, or whitespace.
Preset names are not supported, and in-game effects may override the setting.

## Validation and Limits

Duplicate keys and invalid values are rejected. Invalid settings do not
replace an existing output file.

The NSD sound-effects, music-volume, ambient-volume, and effects-flags fields
are not exposed as settings and remain zero. Their authoring meanings and
units are not established. The game's separate profile music-volume control
does not establish how to use these scene fields.

Native checks confirm field loading and the fog, camera-distance, and reverb
handoffs in the examined debug build. They do not verify visible rendering,
audible output, asset availability, or complete level loading.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NSD](../formats/nsd.md) | [Level relationships](./level-relationships.md) | [NIL lighting](./nil-lighting.md) | [NIL collision](./nil-collision.md)
