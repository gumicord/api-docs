# Manifest and tokens

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

An `abi` above the client's warns and applies only known rules.
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

```jsonc
{ "background": "$color.bg.raised" }
{
  "color.brand": "#7c6cf0",
  "color.accent": "$color.brand"
}
```

Lengths are always logical px. Conversion to physical px belongs to the
renderer.

## `$data.tint`

Not a token: a mark for color the node brought (role color, folder
color). Writable in `color`, `borderColor`, `background` only. Nodes
without color ignore it; a later plain color wins. No identifier crosses
(one server cannot be singled out).

```jsonc
// Default color. Stops here when the person carries no color
{ "select": "nav.member_list.item.name", "style": { "color": "$color.text.secondary" } },
// Only color carriers take this color
{ "select": "nav.member_list.item.name", "style": { "color": "$data.tint" } }
```

| | |
|---|---|
| Writable in | `color` / `borderColor` / `background` |
| Nodes without color | Nothing happens. The earlier rule's value stays |
| A later plain color rule | That one wins. The mark drops |

Only `tint`-carrying nodes carry color. Currently
`nav.guild_list.folder` / `nav.guild_list.folder.icon` (folder color) and
`nav.member_list.item.name` (role color).
