# NSA Material Properties

The NSA format is the central hub connecting geometry to textures and surface behavior. Each material entry contains:

| Property | Type | Meaning |
|----------|------|---------|
| `texture` | string | Primary diffuse texture name (→ DDS) |
| `texture2` | string | Secondary/alternate texture name (→ DDS) |
| `glow` | string | Emissive overlay map name (→ DDS) |
| `detail` | string | Tiled detail texture name (→ DDS) |
| `envmap` | string | Environment reflection map name (→ DDS) |
| `shaderType` | string | Xbox shader program identifier |
| `intensity` | float | Glow intensity scalar |
| `detailscale` | float | Detail UV tiling factor |
| `pageflip` | int | Animated texture frame count |
| `blendMode` | string | Alpha blending mode |
| `alphaTest` | string | Alpha test enable/threshold |

## Xbox Shader Types in NSA

The `shaderType` property identifies which Xbox vertex/pixel shader program pair the original engine used. These define the intended surface behavior:

| shaderType value(s) | Xbox shader source | Surface behavior |
|---|---|---|
| *(none/default)* | `default*.xvu` | Standard lit diffuse |
| `glossenvmap`, `detail`, `bumpenv`, `envmap`, `boxproject` | `Envmap*.xvu + bumpreflect.ps` | Specular + environment reflection |
| `glow`, `scanline` | `selfglow.xpu` | Self-illuminating emissive |
| `fullbright`, `sky`, `add`, `halo`, `lightvolume`, `pageflip` | `fullbright*` | Unlit texture color |
| `ghostsuit`, `cloak` | `ghostsuit.xpu + cloak*.xvu` | Fresnel rim + cloaking transparency |
| `glass`, `etchedglass`, `etchedmasked`, `fog`, `fogcolor` | `glass*` | Transparent glass + Fresnel edge |
| `bump` | `bump*.xvu + emBump.ps` | Normal/bump mapped |
| `metalgloss`, `metalenvmap` | `MetalEnvmap*.xvu` | Metallic environment reflection |
| `membrane`, `shield`, `psishieldhit` | `membrane*.xvu + shield.ps` | Energy shield effect |
| `lava` | `lava.xpu` | Scrolling UV + emissive glow |
| `water`, `bumpwater` | `waterNM*.xvu + waterNM.ps` | Animated water surface |
| `alphadissolve`, `dissolveglow` | `alphadissolve.ps` | Noise-based alpha dissolve |

## Blend Mode Classification

The `shaderType` value also implies the intended transparency mode:

| Blend mode | shaderType triggers |
|---|---|
| **Opaque** | default, most standard materials |
| **Masked** | `masked`, `etchedmasked` |
| **Additive** | `glow`, `add`, `fullbright`, `halo`, `lava`, names ending in `_glo` |
| **Alpha blended** | `alpha`, `glass`, `water`, `membrane`, `fog` |
| **Not rendered** | `sky`, `portal`, `ge_shadow`, `invis`, `video`, names ending in `_bink` |

---

[Documentation home](../../README.md) | [Implementation guides](../IMPLEMENTATION.md)

Related: [NSA](../formats/nsa.md) | [DDS](../formats/dds.md) | [Dependency graph](./dependency-graph.md) | [Xbox shader types](../reference/xbox-shader-types.md)
