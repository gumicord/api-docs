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

## Manifest

| Field | Required | Shape |
|---|---|---|
| `id` | required | Reverse domain (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`) |
| `name` | required | 1–64 chars. Display name |
| `version` | required | Semantic version (`X.Y.Z`, prerelease allowed) |
| `abi` | required | Integer ≥ 1. Assumed UITree ABI major version |
| `author` | optional | ≤ 128 chars |
| `description` | optional | ≤ 512 chars |
| `homepage` | optional | URI |
| `license` | optional | ≤ 64 chars |
| `remoteAssets` | optional | External asset hostnames (≤ 16). Undeclared hosts are never fetched |

An `abi` above the client's warns the user and applies only known rules.
At or below applies normally.

## Tokens

A reusable value table. Free names (`category.use.variant` recommended).
The client never requires names.

| Value type | Shape | Example |
|---|---|---|
| color | `#RGB` / `#RRGGBB` / `#RRGGBBAA` | `"#7c6cf0"`, `"#ffffff14"` |
| length | Number, logical px (renderer scales for DPI) | `8` |
| font | Object | `{ "family": "Inter", "size": 15, "lineHeight": 22, "weight": 600 }` |
| shadow | Object | `{ "x": 0, "y": 2, "blur": 8, "color": "#00000040" }` |
| time | Number, milliseconds | `150` |

Font fields: `family` (omitted uses the bundled default; omitting is
preferred), `size`, `lineHeight`, `weight` (100–900), `italic`,
`letterSpacing`. Shadow fields: `x`, `y`, `blur`, `spread`, `color`
(required).

Reference with `$name`. Tokens may reference tokens; cycles error.

### `$data.tint`

Not a token: a mark for color the node brought (role color, folder
color). Writable in `color`, `borderColor`, `background` only. Nodes
without color ignore it; a later plain color wins. No identifier crosses
(one server cannot be singled out).

## Rules

`select` and `style` required, `when` optional, nothing else.
Application is in written order, per property (later wins). No specificity.

- `select` is an exact stable-ID match. No wildcards. Find IDs in the
  client's stable-ID table (`chat.message.content` and friends).

### `when`

AND across keys; `state` arrays are AND (all must hold), platform arrays
are OR (any may hold).

| Key | Shape |
|---|---|
| `state` | Name or array. `hover`・`active`・`focus`・`selected`・`disabled`・`unread`・`mentioned`・`loading`・`grouped`・`collapsed` |
| `platform` | `windows`・`macos`・`linux`・`android`・`ios`・`desktop`・`mobile`, or array |
| `colorScheme` | `light`・`dark` (follows the OS) |
| `minWidth`・`maxWidth` | Number, inclusive window bounds in logical px |
| `slot` | String. Positional distinction for same-ID siblings only; never matches snowflakes (keeps themes shareable and private) |

Example: `{ "select": "nav.guild_list.item", "when": { "platform": "mobile" },
"style": { "width": 40, "height": 40 } }`.

A rule with an unknown key, state, or platform name drops whole
(dropping just the condition would widen it).

## Styles

Syntax, defaults, inheritance, and interpolation per property. Unknown
properties warn and drop (forward compatibility).

| Property | Value shape | Default, inheritance, notes |
|---|---|---|
| `background` | Color, background object, or `$` ref | Default transparent. Composites color → image → tint, bottom up |
| `color` | Color, `$` ref, or `$data.tint` | Inherits to children. Text color |
| `font` | Font or `$` ref | Inherits to children |
| `borderColor` | Color, `$` ref, or `$data.tint` | Border color |
| `borderWidth` | Length or `$` ref | Border thickness |
| `radius` | Length or `$` ref | Corner rounding. Interpolates |
| `padding`・`margin` | Length or `[top, right, bottom, left]` | Inner, outer spacing |
| `gap` | Length or `$` ref | Spacing between children. Interpolates |
| `width`・`height` | Length or `$` ref | Interpolates (surrounding layout moves too) |
| `minWidth`・`maxWidth`・`minHeight`・`maxHeight` | Length or `$` ref | Constraints (not the `when` namesakes) |
| `opacity` | 0.0–1.0 | Interpolates |
| `shadow` | Shadow or `$` ref | Switches instantly |
| `transition` | Milliseconds | Time to chase new values. Draws nothing itself |
| `decoration` | Space-separated `none`・`underline`・`strikethrough` | One unknown word drops the property |

Only `color` and `font` inherit. Layout overrides are M2; M1 themes
change appearance only.

### `transition`

Animates colors (`color`, `borderColor`, `background` color), `radius`,
`borderWidth`, `opacity`, sizes, `gap`. Fonts, shadows, and images
switch instantly. Ease-out only, no inheritance, only the written node
moves. Never blends against an unspecified side. Time-driven, so 60Hz
and 144Hz agree.

### Inline decoration slots

On `primitive.text` via `when.slot`: `bold` (`**`), `italic` (`*`),
`underline` (`__`), `strike` (`~~`), `spoiler` (`||`; hidden uses
`color` as fill), `code` (`` ` ``), `link`, `mention`, `h1`, `h2`, `h3`,
`subtext`, `bullet`, `quote_bar`, `li0`–`li4` (list depth; the theme
decides the px width). Overlaps stack in written order.

## Backgrounds and assets

`image` takes `fit` (`cover` default, `contain`, `stretch`, `tile`,
`none`), `position` (two 0.0–1.0 anchors, default center), `opacity`
(default 1.0), `blur` (applied once at load, max 256). Any stable ID may
carry one. To show a full-window picture, leave `chat.message_list` and
`chat.view` uncolored (transparent shows the window picture through).

Three reference forms:

- Theme-relative paths. `assets/` recommended. No `../` escape.
  `png|jpg|jpeg|webp|avif|woff2|ttf|otf` only.
- `data:` embeds. Those image types plus `font/woff2` only.
- `https:` externals. Need `remoteAssets` declaration plus user approval.
  Denial and failures fall back to `background.color` (the theme keeps
  going).

No credentials sent, no cross-host redirect chase, fetched once then
cached, 32MB per file.

## Breakage

Only broken JSON syntax and invalid manifests discard everything. All
else drops by spot:

| State | Drops | Told as |
|---|---|---|
| Token cycles, undefined refs, mistyped values | That property (cycles: referring rules) | error |
| Unknown stable IDs, properties, conditions | Rule or spot | warning (forward compat) |
| Unfetchable assets | `background.color` fallback | warning ("unapproved" for denials) |

Listed in settings with positions (JSON paths). Never blocks startup.

## Selection and updates

- Selection records in `themes/active.json` (`{"theme": "<manifest id>"}`).
  Missing means the standard theme.
- Standard is bundled Midnight (no file, excluded from hot reload).
- A `GUMICORD_THEME` file wins (hidden from the list).
- Unreadable themes are unlisted, unapplied; the previous theme stays
  with notice.
- File changes reapply without restart. Mid-edit breakage keeps the last
  good theme.
