# Theme API

## Getting started

A theme is one JSON file changing how the client looks. No CSS (no
parser, no specificity to learn).

To get going:

1. Make one folder under `themes/` (`mytheme/`) holding `theme.json`.
2. Copy the minimal example below.
3. While developing, swap it in with
   `GUMICORD_THEME=themes/mytheme/theme.json`.
4. Saving the file reapplies without restart. Mid-edit breakage keeps
   the last good theme.

Minimal example:

```jsonc
{
  "$schema": "https://gumicord.dev/schema/theme-1.json",
  "manifest": { "id": "dev.example.mytheme", "name": "My theme",
                "version": "1.0.0", "abi": 1 },
  "tokens": {
    "color.bg.base": "#101018",
    "color.text.primary": "#f0f0f5"
  },
  "rules": [
    { "select": "app.window", "style": { "background": "$color.bg.base" } },
    { "select": "chat.message.content",
      "style": { "color": "$color.text.primary" } },
    { "select": "chat.message",
      "when": { "state": "hover" },
      "style": { "background": "#ffffff14" } }
  ]
}
```

Top level holds only `$schema`, `manifest`, `tokens`, `rules`.

## Contents

- [Setup and authoring](en/theme/setup.md) — tooling, first steps, samples
- [Stable ID catalog](en/theme/ids.md) — IDs writable in `select`, with roles
- [Manifest and tokens](en/theme/manifest-tokens.md) — identity, value table, `$data.tint`
- [Rules and conditions](en/theme/rules.md) — `select`, `when`, cascade, selection
- [Styles](en/theme/styles.md) — properties, `transition`, inline slots
- [Backgrounds, assets, breakage](en/theme/assets.md) — images, reference forms, validation
