# Backgrounds, assets, breakage

## Background object

Beside `image` it takes `fit`, `position`, `opacity`, `blur`. Any stable
ID may carry one.

| Key | Type | Default | Meaning |
|---|---|---|---|
| `color` | Color | Transparent | Color under the image. Also the fallback when the image cannot load |
| `image` | Asset reference | — | Background image |
| `fit` | `cover` / `contain` / `stretch` / `tile` / `none` | `cover` | How it fits the area |
| `position` | `[x, y]` (each 0.0–1.0) | `[0.5, 0.5]` | Anchor position |
| `opacity` | 0.0–1.0 | `1.0` | Image opacity |
| `blur` | Number (logical px) | `0` | Blur radius. Applied once at load, then cached |
| `tint` | Color | Transparent | Color over the image. Keeps text readable |

Composite order, bottom up: `color` → `image` → `tint`.

To show a full-window picture, leave `chat.message_list` and
`chat.view` uncolored (transparent shows the window picture through).

```jsonc
{
  "select": "app.window",
  "style": {
    "background": {
      "color": "#0f0f17",
      "image": "assets/wallpaper.png",
      "fit": "cover",
      "tint": "#0f0f1740"
    }
  }
}
```

For translucent layering, make the upper UI translucent with 8-digit
colors (`#RRGGBBAA`):

```jsonc
{ "select": "app.window",      "style": { "background": { "image": "assets/bg.png", "fit": "cover" } } },
{ "select": "nav.channel_list","style": { "background": "#16161fcc" } },
{ "select": "chat.header",     "style": { "background": "#0f0f1799" } }
```

## Three reference forms

| Kind | Shape | Example |
|---|---|---|
| Bundled asset | Relative path from `theme.json` | `"assets/wallpaper.png"` |
| Data URI | `data:` | `"data:image/png;base64,..."` |
| External URL | `https:` only | `"https://cdn.example.com/bg.png"` |

- Theme-relative paths. `assets/` recommended. No `../` escape, no
  absolute paths, no symlink chase. `png|jpg|jpeg|webp|avif|woff2|ttf|otf`
  only.
- `data:` embeds. Those image types plus `font/woff2` only.
- `https:` externals. Need `remoteAssets` declaration plus approval.
  Denial and failures fall back to `background.color` (the theme keeps
  going).

```
midnight/
├─ theme.json
└─ assets/
   ├─ wallpaper.png
   └─ Inter.woff2
```

Fetch constraints: no credentials sent, `https` only, fetched once then
cached, 32MB per file, no cross-host redirect chase. Bundled assets are
recommended.

## Breakage

Only broken JSON syntax and invalid manifests discard everything. All
else drops by spot:

| State | Drops | Told as |
|---|---|---|
| Token cycles, undefined refs, mistyped values | That property (cycles: referring rules) | error |
| Unknown stable IDs, properties, conditions | Rule or spot | warning (forward compat) |
| Unfetchable assets | `background.color` fallback | warning ("unapproved" for denials) |

Listed in settings with positions (JSON paths, e.g.
`rules[3].style.background.image`). Never blocks startup.
