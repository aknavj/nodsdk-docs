# Xbox Shader Types

Materials reference Xbox vertex/pixel shader programs identified by filename in `.nsa` properties. The `shaderType` NSA property maps to these original Xbox shader source files:

| shaderType value(s) | Xbox source files | Surface behavior |
|---|---|---|
| *(default)* | `default*.xvu` | Standard lit diffuse |
| `glossenvmap`, `envmap`, `detail`, `bumpenv` | `Envmap*.xvu + bumpreflect.ps` | Specular + environment reflection |
| `glow`, `scanline` | `selfglow.xpu/.ps` | Self-illuminating emissive |
| `fullbright`, `sky`, `add`, `halo`, `pageflip` | `fullbright*` | Unlit texture color |
| `ghostsuit`, `cloak` | `ghostsuit.xpu/.ps + cloak*.xvu` | Ghost suit Fresnel rim |
| `glass`, `etchedglass`, `fog`, `fogcolor` | `glass*` | Transparent glass + Fresnel |
| `bump` | `bump*.xvu + emBump.ps` | Normal/bump mapping |
| `metalgloss`, `metalenvmap` | `MetalEnvmap*.xvu` | Metallic environment reflection |
| `membrane`, `shield`, `psishieldhit` | `membrane*.xvu + shield.ps` | Energy membrane/shield |
| `lava` | `lava.xpu/.ps` | Scrolling lava + glow |
| `water`, `bumpwater` | `waterNM*.xvu + waterNM.ps` | Animated water |
| `alphadissolve`, `dissolveglow` | `alphadissolve.ps` | Alpha dissolve transition |

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NSA format](../formats/nsa.md) | [Material properties and blend modes](../implementation/material-properties.md)
