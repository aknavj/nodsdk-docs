# NUI — Nihilistic UI Layout

**Extension:** `.nui`
**Type:** Text

UI screen layout definitions. 5 files exist: `console`, `debug`, `LoadScreen`, `mainMenu`, `TitleScreen`.

## Format

```
; Comment
name=ScreenName
pos=x,y
dialog=1
music=soundName

element <type> name=... pos=x,y size=w,h color=r,g,b,a font=...
quad name=... pos=x,y size=w,h texture=...
```

Element types define buttons, text fields, images, and other UI widgets.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [DDS](./dds.md)
