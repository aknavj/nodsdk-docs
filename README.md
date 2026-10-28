# nodsdk-docs
OpenNOD SDK community documentation for Nihilistic StarCraft Ghost

## Documentation

- [Asset format reference](./docs/FORMAT_REFERENCE.md): binary layouts and text
  syntax, with a separate page for each of the 23 formats.
- [Implementation guides](./docs/IMPLEMENTATION.md): format dependencies, asset
  naming and lookup, level and model relationships, vertex data, animation,
  materials, and templates.
- [Debug tools](./docs/game/debug-tools.md): enabling the in-game debug menu,
  diagnostic overlays, and reverse-engineering findings.
- Shared reference:
  - [Coordinate system](./docs/reference/coordinate-system.md)
  - [Xbox shader types](./docs/reference/xbox-shader-types.md)
  - [Asset file counts](./docs/reference/asset-file-counts.md)

## Documentation Organization

Keep documentation focused on individual topics:

- `docs/formats/`: one page per asset format, including layouts and examples.
- `docs/implementation/`: one guide per relationship or implementation topic.
- `docs/game/`: in-game behavior and debugging guides.
- `docs/reference/`: shared conventions and asset inventory.

Add new pages to the relevant index and link related formats and guides rather
than expanding the indexes into combined reference documents.
