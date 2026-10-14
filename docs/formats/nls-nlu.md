# NLS/NLU — Nihilistic Localization Strings

**Extension:** `.nls`, `.nlu`
**Type:** Text (key-value)

Localized string tables mapping string identifiers to display text.

## Format

```
# Comment line
STRING_KEY    The displayed text value
ANOTHER_KEY   Another string value
```

Lines beginning with `#` are comments. Keys and values are separated by whitespace. The parser provides case-insensitive key lookup via `LookupString()`.

---

[Documentation home](../../README.md) | [Format reference](../FORMAT_REFERENCE.md)

Related: [NLT](./nlt.md)
