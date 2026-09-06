# `ctx.data` and node shape

## `ctx.data`

Per-patch context `{ data }`. Per-node domain facts typed from the ID
(`ctx.data.author.bot`); `undefined` where the node carries none.
Read-only; no raw payloads ever surface. Additions are non-breaking;
removals and renames are breaking (part of the ABI).

| Type | Main fields |
|---|---|
| `UserData` | `id`, `username`, `displayName`, `bot`, `avatarUrl?` |
| `MessageData` | `id`, `channelId`, `guildId?`, `createdAt`, `editedAt?`, `content` (plain; decorated body lives in nodes), `author`, `pinned`, `referencedMessageId?` |
| `GuildData` | `id`, `name`, `iconUrl?`, `unread`, `mentionCount` |
| `ChannelData` | `id`, `name`, `type`, `topic?`, `nsfw`, `unread`, `mentionCount` |
| `CategoryData` | `id`, `name`, `collapsed` |
| `DmData` | `id`, `recipients`, `unread`, `mentionCount` |
| `MemberData` | `user`, `displayName`, `status` (`online`/`idle`/`dnd`/`offline`), `roles` (names, not IDs) |
| `AttachmentData` | `id`, `filename`, `size`, `contentType?`, `url`, `width?`, `height?` |
| `EmbedData` | `type`, `title?`, `description?`, `url?`, `color?` |

Which ID carries which type lives in the `data` column of the
[stable ID catalog](en/theme/ids.md).

## Node shape (`UINode`)

```ts
interface UINode {
  id: NodeId | PluginNodeId;  // stable ID
  readonly key?: string;      // tells same-ID siblings apart. Read-only
  readonly states?: readonly NodeState[];  // currently held
  readonly tint?: string;     // data-carried color (#RRGGBB). Where it lands is the theme's call
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

`NewUINode` is the same with creatable IDs only. `key`, `states`, and
colors inherit from the input tree and cannot be rewritten (read, never
take home).

`NodeState` is `hover`, `active`, `focus`, `selected`, `disabled`,
`unread`, `mentioned`, `loading`, `grouped`, `collapsed`.

## Host boundary

Only structure and draw content cross. The JS side is never trusted;
returns validate per section.

- Incoming `content` per kind: `text` string, `icon` name, `image` URL,
  `qr` value, `rich` runs (`text` plus `font`, `color {r,g,b,a}`,
  `under`, `through`, `hidden`, `revealed`, `link`, `image`), `editable`
  field (`text`, `caret`, `selection {start,end}`, `composing`,
  `placeholder`).
- `props` backfills only when `content` is absent, from its first
  `value`/`text`/`label` or `name`/`url`.

| Target | `props` | Draw content |
|---|---|---|
| `primitive.text`, `primitive.badge`, `primitive.button`, `plugin.*` | First string among `value` / `text` / `label` | That string |
| `primitive.icon` | `name` string | Picture of that name |
| `primitive.image` | `url` string | Picture of that URL |
| Else | — | No draw content (children still draw) |

Functions like `onPress` cannot cross the boundary and drop silently.
Theme-owned specs like `tone` or `gap` drop too. Neither warns — a
correct plugin warning every frame would make logs unreadable.
