# Styles

Syntax, defaults, inheritance, and interpolation per property. Unknown
properties warn and drop (forward compatibility).

| Property | Value shape | Default, inheritance, notes |
|---|---|---|
| `background` | Color, background object, or `$` ref | Default transparent. Composites color → image → tint, bottom up. See [Backgrounds and assets](en/theme/assets.md) |
| `color` | Color, `$` ref, or `$data.tint` | Inherits to children. Text color |
| `font` | Font or `$` ref | Inherits to children |
| `borderColor` | Color, `$` ref, or `$data.tint` | Border color |
| `borderWidth` | Length or `$` ref | Border thickness |
| `radius` | Length or `$` ref | Corner rounding. Interpolates |
| `padding`, `margin` | Length or `[top, right, bottom, left]` | Inner, outer spacing |
| `gap` | Length or `$` ref | Spacing between children. Interpolates |
| `width`, `height` | Length or `$` ref | Interpolates (surrounding layout moves too) |
| `minWidth`, `maxWidth`, `minHeight`, `maxHeight` | Length or `$` ref | Constraints (not the `when` namesakes) |
| `opacity` | 0.0–1.0 | Interpolates |
| `shadow` | Shadow or `$` ref | Switches instantly |
| `transition` | Milliseconds | Time to chase new values. Draws nothing itself |
| `decoration` | Space-separated `none`, `underline`, `strikethrough` | One unknown word drops the property |

Only `color` and `font` inherit. Layout overrides are M2; M1 themes
change appearance only.

## `transition`

Animates colors (`color`, `borderColor`, `background` color), `radius`,
`borderWidth`, `opacity`, sizes, `gap`. Fonts, shadows, and images
switch instantly. Ease-out only, no inheritance, only the written node
moves. Never blends against an unspecified side. Time-driven, so 60Hz
and 144Hz agree.

```jsonc
{ "select": "nav.guild_list.item.icon",
  "style": { "radius": 8, "transition": 150 } },
{ "select": "nav.guild_list.item.icon",
  "when": { "state": "hover" }, "style": { "radius": 4 } }
```

Only the written node moves. Moving everything by default turns every
list scroll into dozens of moving rows. Moving sizes moves surrounding
layout too.

## Inline decoration slots

On `primitive.text` via `when.slot`. Overlaps stack in written order.

| Slot | When |
|---|---|
| `bold`, `italic`, `underline`, `strike` | `**`, `*`, `__`, `~~` |
| `spoiler` | `||`; hidden uses `color` as fill |
| `code` | `` ` `` (inline) |
| `link`, `mention` | Links, bare URLs / `<@1>`, `<#1>`, `<@&1>`, `@everyone` |
| `h1`, `h2`, `h3`, `subtext` | Headings and `-# ` |
| `bullet` | List marker |

```json
{ "select": "primitive.text", "when": { "slot": "underline" },
  "style": { "decoration": "underline" } },
{ "select": "primitive.text", "when": { "slot": "strike" },
  "style": { "decoration": "strikethrough", "color": "$color.text.muted" } }
```

Stacked things take stable IDs as usual: quote lines are
`primitive.divider` with `slot: "quote_bar"`, code blocks are
`primitive.code_block`, list indents are `layout.row` with
`slot: "li0"`–`"li4"` (the theme decides the px width).
