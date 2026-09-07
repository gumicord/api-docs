# Rules and conditions

`select` and `style` required, `when` optional, nothing else.
Application is in written order, per property (later wins). No specificity.

Find IDs in the [stable ID catalog](en/theme/ids.md).

## `select`

Exact stable-ID match only. No wildcards or descendant selectors.

```jsonc
{ "select": "chat.message.header.author", "style": { /* ... */ } }
```

Pattern matching is withheld so added IDs never restyle existing themes.
Forward compatibility wins.

## `when`

AND across keys; `state` arrays are AND (all must hold), platform arrays
are OR (any may hold).

| Key | Shape |
|---|---|
| `state` | Name or array. `hover`, `active`, `focus`, `selected`, `disabled`, `unread`, `mentioned`, `loading`, `grouped`, `collapsed` |
| `platform` | `windows`, `macos`, `linux`, `android`, `ios`, `desktop`, `mobile`, or array |
| `colorScheme` | `light`, `dark` (follows the OS) |
| `minWidth`, `maxWidth` | Number, inclusive window bounds in logical px |
| `slot` | String. Positional distinction for same-ID siblings only; never matches snowflakes (keeps themes shareable) |

Examples:

```jsonc
{ "select": "nav.guild_list.item", "when": { "platform": "mobile" },
  "style": { "width": 40, "height": 40 } }
{
  "select": "nav.channel_list.item",
  "when": { "state": ["hover", "unread"], "colorScheme": "dark" },
  "style": { /* ... */ }
}
```

States can stand together, so an array means "both". Platforms hold one
at a time, so an array means "either". Each takes the only reading that
makes sense.

`slot` exposes position only. Matching `Id` (snowflake) would allow
"paint only this server", which stops themes from being shareable.
`Index` points elsewhere once the order changes, so it never decorates.

```json
{ "select": "nav.user_panel.presence", "when": { "slot": "dnd" },
  "style": { "background": "#e05260" } }
```

A rule with an unknown key, state, or platform name drops whole
(dropping just the condition would widen it).

## Cascade

One rule only: **apply in written order; later rules overwrite earlier
ones.**

```jsonc
"rules": [
  { "select": "chat.message", "style": { "background": "#111" } },
  { "select": "chat.message", "when": { "state": "hover" }, "style": { "background": "#222" } }
]
// #222 while hovering. Reversed order silences hover.
```

Overwriting happens per property:

```jsonc
{ "select": "chat.message", "style": { "background": "#111", "radius": 8 } },
{ "select": "chat.message", "style": { "background": "#222" } }
// Result: background=#222, radius=8
```

Written order is everything, so reading top-down always explains why a
rule loses.

Plugins apply after themes. On collision the plugin wins.

## Selection

- Selection records in `themes/active.json` (`{"theme": "<manifest id>"}`).
  Missing means the standard theme.
- Standard is bundled Midnight (no file, excluded from hot reload).
- A `GUMICORD_THEME` file wins (hidden from the list).
- Unreadable themes are unlisted, unapplied; the previous theme stays
  with notice.
- File changes reapply without restart. Mid-edit breakage keeps the last
  good theme.
