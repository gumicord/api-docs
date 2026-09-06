# Setup and authoring

## Tooling

No dedicated tooling is needed. A text editor is enough. No Node.js, no
build step (`theme.json` is read as-is).

Nice to have:

- A JSON Schema–aware editor. Writing
  `"$schema": "https://gumicord.dev/schema/theme-1.json"` at the top
  enables completion and checking (optional at runtime).

## Layout

Make one folder under `themes/` holding `theme.json`. Bundled pictures
and fonts go into `assets/` in the same folder.

```
themes/
└─ mytheme/
   ├─ theme.json
   └─ assets/
      └─ wallpaper.png
```

## Dev loop

1. Start swapped in with `GUMICORD_THEME=themes/mytheme/theme.json`
   (a dev override, hidden from the list).
2. Saving reapplies without restart.
3. Mid-edit breakage keeps the last good theme. Broken spots appear in
   the settings list with JSON paths.
4. Once settled, pick it from the theme list in settings. Selection
   records in `themes/active.json` (`{"theme": "<manifest id>"}`).

## Authoring

Start small. What is unwritten stays standard.

1. Write the identity in `manifest` (`id` is world-unique. Reverse
   domain).
2. Name reusable colors and sizes in `tokens` (paint dishes).
3. Name parts by stable ID (`select`) and write `style`. Look IDs up in
   the [stable ID catalog](en/theme/ids.md).
4. Add `when` for conditions (`state`, `platform`, and friends). See
   [Rules and conditions](en/theme/rules.md).
5. Save and look at the screen.

## Sample

Full example (read line by line):

```jsonc
{
  "$schema": "https://gumicord.dev/schema/theme-1.json",  // completion aid. Runs without it
  "manifest": { "id": "dev.example.mytheme", "name": "My theme",
                "version": "1.0.0", "abi": 1 },  // identity
  "tokens": {
    "color.bg.base": "#101018",      // window ground color
    "color.bg.hover": "#ffffff14",   // row hover light
    "color.text.primary": "#f0f0f5", // body text color
    "color.accent": "#7c6cf0"        // accent color
  },
  "rules": [
    // Ground the whole window. Pull tokens by $name
    { "select": "app.window", "style": { "background": "$color.bg.base" } },
    // Body text in the primary color
    { "select": "chat.message.content",
      "style": { "color": "$color.text.primary" } },
    // Author names in the accent color
    { "select": "chat.message.header.author",
      "style": { "color": "$color.accent" } },
    // Light only the hovered row (conditional example)
    { "select": "chat.message",
      "when": { "state": "hover" },
      "style": { "background": "$color.bg.hover" } },
    // Small round server column on narrow screens (mobile example)
    { "select": "nav.guild_list.item",
      "when": { "platform": "mobile" },
      "style": { "width": 40, "height": 40 } }
  ]
}
```

What this changes: window ground color, body and author text colors,
row hover light, server column size on mobile. The rest stays standard.

Read next: [Manifest and tokens](en/theme/manifest-tokens.md),
[Styles](en/theme/styles.md), [Backgrounds and assets](en/theme/assets.md).
