# Manifest, capabilities, approval

## `manifest.json`

No unknown top-level keys. `$schema` is
`https://gumicord.dev/schema/plugin-manifest-1.json`.

| Field | Required | Shape |
|---|---|---|
| `id` | required | Reverse domain (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`). Matches the directory name |
| `name` | required | 1–64 chars. Display name |
| `version` | required | Semantic version (`X.Y.Z`, prerelease allowed) |
| `entry` | optional | Flat `.js` / `.qjsc` name, defaults to `plugin.js` |
| `description` | optional | ≤ 512 chars. Shown on the approval screen |
| `capabilities` | optional | Deduped array. `"log"` and `"storage"` only; unknown names fail the load |
| `settings` | optional | Flat `.js` / `.qjsc`. Omitted means no settings screen |

## Capabilities

Only declared ones are injected. Calling an undeclared API throws
`TypeError` (treated as absent, not denied).

| Capability | Grants |
|---|---|
| `log` | `log.info` / `warn` / `error` |
| `storage` | `storage.get` / `set` / `remove` |

`ui.exists` and failure reporting are always injected, not permissions.

`log` capability logging:

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

Effect: lands in the host log. Argument is the message. Returns nothing.

`storage` is a small host-side store. Split per plugin, survives reloads:

```ts
get: (key: string) => string | null
set: (key: string, value: string) => void
remove: (key: string) => void
getJSON<T>(key: string, fallback: T): T
setJSON(key: string, value: unknown): void
```

- Arguments: keys and values. `getJSON`'s `fallback` returns for both
  missing and broken JSON.
- Returns: `get` yields `null` when missing. The rest return nothing.
- Effect: `set`/`remove` persist immediately.
- Limit: never call from inside a patch (persist from the settings side).

## Approval and states

First-seen permissions wait for approval. Grants and denials persist.
Empty means denied, missing key means unconfirmed (different things).
Disabling stops while keeping the grant; re-enabling does not re-ask.
Denials and revokes live in settings.

Denied plugins never load. Running short on permissions would pile up
broken patches and waste the disable counter.

## Settings screens

Render into `settings.*`. No actionable widgets yet. Persist from the
settings side.

```ts
settings(fn: () => UINode): void
```

Registers a display-only settings page. Effect: the page shows in settings.

- Argument: node factory, called at display time.
- Returns: nothing.
- Limit: no actionable widgets yet (their event channel arrives
  separately). Persist from the settings side, never from a patch.
