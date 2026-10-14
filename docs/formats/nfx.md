# NFX — Nihilistic Effects

**Extension:** `.nfx`
**Type:** Text (block-structured)

Particle system and visual effects definitions. The main `game.nfx` contains ~1100+ effect blocks and ~144 controllers.

## Format

```
; Semicolon comment
<blockType> <name>
    property1 value1
    property2 value2
end
```

## Block Types

`controller`, `smoke`, `sparks`, `glow`, `light`, `shockwave`, `spikes`, `model`, `fxmodel`, `decal`, `trail`, `billboard`, `damage`, `ctrldamage`, `force`, `spawner`, `mushroom`, `volumefog`, `energybeam`, `rain`, `muzzle_flash`, `exp_fire`, `project`, `projectcube`

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NLFX](./nlfx.md) | [Level Format Relationships](../implementation/level-relationships.md)
