# Ordered Templates and Static Transforms

NOT templates and NSD placements share counted arrays of 36-byte modifiers.
They are not single-parent tree nodes. The [NOT format](../formats/not.md)
describes the binary layout, archive registry, and verified metadata types.

## Two Different Precedence Rules

- **Template registration:** load `Global.not` first, then the scene's named archives in
  order. The first registered duplicate template name wins.
- **Modifier values:** expand each `basedon` in place and process all entries in
  source order. A later same-ID value replaces the earlier value in place.

For example, these are symbolic modifier streams, not serialized text:

```
TemplateA: model = "model_a.nod"
TemplateB: model = "model_b.nod"

Placement:
  basedon = TemplateA
  model   = "local.nod"
  basedon = TemplateB

Resolved model: "model_b.nod"
```

Index all selected archives before resolving references. Reject missing
references, cycles, and more than 128 expanded modifiers without publishing a
partial result. A local model override cannot repair an earlier missing
reference. Preserve explicitly empty model values.

## Resolved Values and Native Dispatch

The legacy NSD `type_name` field names a template. Runtime modifier 20 determines
native thing construction; modifier 4 supplies the model. Do not derive either
from class-name heuristics or guessed model aliases. ID 15 supplies a script
reference; IDs 46 (`CASTID`) and 161 (`EFFECT1`) are different string properties.

## Static Model Transforms

The model-creation and modifier paths establish these order-sensitive rules:

1. ID 4 creates a model instance and resets its uniform scale to 1.
2. ID 143 (`MODELSCALE`) applies only after an instance exists. An earlier scale
   is ignored. Native zero and negative scales are accepted.
3. ID 174 (`MODEL_LODSCALE`) changes LOD selection, not vertex geometry.
4. ID 175 (`MODELOFFSET`) affects geometry only when ID 12 (`renderflags`) has
   bit `0x2000`. The enabled offset is added directly to world translation,
   without rotation or scaling:

```
world = origin + enabled_offset + rotation * (scale * model_vertex)
```

The model constructor does not explicitly initialize the offset. The
reconstructed static evaluator rejects an enabled offset without a value,
rather than guessing zero. Inactive offset values do not affect geometry, and
template names containing `_offset` do not imply any transform.

This describes the static modifier path, not animation, controllers, or later
gameplay mutations. Region bounds and inline meshes have their own
[NSD framing](../formats/nsd.md#inline-region-geometry).

## Native Evidence

In the matching debug executable, relevant paths are object modifiers
(`0x0052EC90`), model creation's scale reset (`0x006884E1`), scale setter
(`0x0052F7D0`), offset setters (`0x0043D870`, `0x0043D8D0`), render flags
(`0x00546CC0`), world-offset addition (`0x0043D080`), and prescaling
(`0x00687172`). These addresses are build-specific evidence, not API constants.

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NOT](../formats/not.md) | [NSD](../formats/nsd.md) | [Dependency graph](./dependency-graph.md)
