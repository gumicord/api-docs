# Plugin API reference

A signature-browsing page. For flows see the concept pages:
[Execution model and lifetime](en/plugins/lifecycle.md),
[Manifest, capabilities, approval](en/plugins/manifest.md),
[`ui`](en/plugins/ui.md),
[`ctx.data` and node shape](en/plugins/data.md).

- There is no event subscription API. See [Events](#events).

## Contents

- [`ui`](#ui): `patch`, `exists`, `wrap`, `after`, `before`, `settings`, `stack`, `node`, `text`, `badge`, `button`, `icon`
- [`log`](#log): `info`, `warn`, `error`
- [`storage`](#storage): `get`, `set`, `remove`, `getJSON`, `setJSON`
- [Interfaces](#interfaces): `UINode`, `NewUINode`, `PatchContext`, `PatchFn`, 8 data types
- [Type aliases and enum-like types](#type-aliases-and-enum-like-types): `NodeId`, `PluginNodeId`, `CreatableNodeId`, `CoreCreatableNodeId`, `NodeState`, `DataByNode`
- [Events](#events): no subscription API

## `ui`

### `ui.patch`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
```

Registers a transform on a stable ID.

- Arguments: `id` (stable ID; type-checked, unknown IDs do not pass),
  `fn` (`(node, ctx) => UINode`; the returned node is used as-is).
- Returns: nothing.
- Effects: bottom-up traversal; children arrive already patched. Matching
  uses the pre-apply ID; output is not recursed into. Same-node patches
  chain in registration order.
- Notes: registering on a missing ID is harmless (never runs).
  Virtualized offscreen nodes are never visited. Keep `fn` pure (call
  counts per message are undefined).

```ts
ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

### `ui.exists`

```ts
exists(id: NodeId): boolean
```

- Argument: `id` (stable ID).
- Returns: `true` when the ID can exist here (`chrome.*` is absent on
  mobile).
- Effects: existence check only. Registering on a missing ID is already
  harmless, so this is only for beforehand branching.

### `ui.wrap`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

- Arguments: wrapped node, parent (no `children` passed).
- Returns: the original under `children: [node]` of a new parent.
- Limit: core IDs (`app.*` / `chrome.*` / `nav.*` / `chat.*`) cannot wrap.
  Own namespace plus `primitive.*` / `layout.*` only.

### `ui.after`

```ts
after(node: UINode, sibling: UINode): UINode
```

- Arguments: target node and sibling.
- Returns: wrapping row node
  (`{ id: "layout.row", children: [node, sibling] }`).
- Effects: adds a sibling after. Output counts as final.

### `ui.before`

```ts
before(node: UINode, sibling: UINode): UINode
```

- Arguments: target node and sibling.
- Returns: wrapping row node
  (`{ id: "layout.row", children: [sibling, node] }`).
- Effects: adds a sibling before. Output counts as final.

### `ui.settings`

```ts
settings(fn: () => UINode): void
```

- Argument: node factory, called at display time.
- Returns: nothing.
- Effects: registers a display-only settings page. The page shows in
  settings.
- Limit: no actionable widgets yet. Persist from the settings side, never
  from a patch.

### `ui.stack`

```ts
stack(nodes: UINode[]): UINode
```

- Argument: nodes to stack.
- Returns: column node (`{ id: "layout.column", children: nodes }`).

### `ui.node`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

- Arguments: `id` (creatable IDs only: `plugin.` + own ID with `.` as `_`,
  or `primitive.*` / `layout.*`), `props`, `children`.
- Returns: new node (`{ id, props?, children? }`).
- Limit: forging a core ID (`app.*` and friends) discards that whole
  output.

### `ui.text`

```ts
text(value: string): NewUINode
```

- Argument: display string.
- Returns: `primitive.text` node (string in `props.value`).

### `ui.badge`

```ts
badge(opts: { text: string; tone?: string }): NewUINode
```

- Argument: badge mark (`text` plus optional `tone`).
- Returns: `primitive.badge` node.
- Note: theme-owned specs such as `tone` drop.

### `ui.button`

```ts
button(opts: { label: string; onPress: () => void }): NewUINode
```

- Argument: label plus press action.
- Returns: `primitive.button` node.
- Note: functions (`onPress`) cannot cross the host boundary and drop
  silently. `button` is inert on settings screens.

### `ui.icon`

```ts
icon(name: string): NewUINode
```

- Argument: picture name.
- Returns: `primitive.icon` node (name in `props.name`).

## `log`

`log` capability logging. Effect: lands in the host log. Argument is the
message. Returns nothing.

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

## `storage`

Small host-side store. Split per plugin, survives reloads.

### `storage.get`

```ts
get: (key: string) => string | null
```

- Argument: key.
- Returns: `null` when missing.

### `storage.set`

```ts
set: (key: string, value: string) => void
```

- Arguments: key and value.
- Effects: persists immediately.
- Limit: never call from inside a patch (persist from the settings side).

### `storage.remove`

```ts
remove: (key: string) => void
```

- Argument: key.
- Effects: persists immediately.
- Limit: never call from inside a patch.

### `storage.getJSON`

```ts
getJSON<T>(key: string, fallback: T): T
```

- Arguments: key and `fallback` (returned for both missing and broken
  JSON).
- Returns: stored value, or `fallback`.

### `storage.setJSON`

```ts
setJSON(key: string, value: unknown): void
```

- Arguments: key and value (stored JSON-encoded).
- Limit: never call from inside a patch.

## Interfaces

### `UINode`

```ts
interface UINode {
  id: NodeId | PluginNodeId;
  readonly key?: string;
  readonly states?: readonly NodeState[];
  readonly tint?: string;
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

- `id`: stable ID.
- `key`: tells same-ID siblings apart. Read-only.
- `states`: currently held. Read-only.
- `tint`: data-carried color (`#RRGGBB`). Where it lands is the theme's
  call. Read-only.
- `props` / `children`: attached values and children.

### `NewUINode`

```ts
interface NewUINode extends UINode {
  id: CreatableNodeId;
}
```

A created node. Same as `UINode` except `id` is limited to creatable IDs.
`key`, `states`, colors carry over from the input tree and cannot be
rewritten.

### `PatchContext`

```ts
interface PatchContext<Id extends NodeId = NodeId> {
  readonly data: Id extends keyof DataByNode ? Readonly<DataByNode[Id]> : undefined;
}
```

Per-patch context. `data` is typed from the ID
(`ctx.data.author.bot`); `undefined` where the node carries none.
Read-only. `Context` is an alias.

### `PatchFn`

```ts
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

A node transform. Must be pure (call counts are undefined).

### `UserData`

```ts
interface UserData {
  readonly id: string;
  readonly username: string;
  readonly displayName: string;
  readonly bot: boolean;
  readonly avatarUrl?: string;
}
```

### `MessageData`

```ts
interface MessageData {
  readonly id: string;
  readonly channelId: string;
  readonly guildId?: string;
  readonly createdAt: string;
  readonly editedAt?: string;
  readonly content: string;
  readonly author: UserData;
  readonly pinned: boolean;
  readonly referencedMessageId?: string;
}
```

`content` is plain text; the decorated body lives in nodes.

### `GuildData`

```ts
interface GuildData {
  readonly id: string;
  readonly name: string;
  readonly iconUrl?: string;
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `ChannelData`

```ts
interface ChannelData {
  readonly id: string;
  readonly name: string;
  readonly type: string;
  readonly topic?: string;
  readonly nsfw: boolean;
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `CategoryData`

```ts
interface CategoryData {
  readonly id: string;
  readonly name: string;
  readonly collapsed: boolean;
}
```

### `DmData`

```ts
interface DmData {
  readonly id: string;
  readonly recipients: readonly UserData[];
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `MemberData`

```ts
interface MemberData {
  readonly user: UserData;
  readonly displayName: string;
  readonly status: string;
  readonly roles: readonly string[];
}
```

`status` is `online` / `idle` / `dnd` / `offline`. `roles` are names, not
IDs.

### `AttachmentData`

```ts
interface AttachmentData {
  readonly id: string;
  readonly filename: string;
  readonly size: number;
  readonly contentType?: string;
  readonly url: string;
  readonly width?: number;
  readonly height?: number;
}
```

### `EmbedData`

```ts
interface EmbedData {
  readonly type: string;
  readonly title?: string;
  readonly description?: string;
  readonly url?: string;
  readonly color?: number;
}
```

## Type aliases and enum-like types

### `NodeId`

Union of stable IDs (121; unknown IDs do not pass the type check). For
the list see the [stable ID catalog](en/theme/ids.md).

```ts
type NodeId = "app.root" | "app.window" | /* ... */ | "layout.scrollbar.thumb";
```

### `PluginNodeId`

```ts
type PluginNodeId = `plugin.${string}`;
```

Own-namespace IDs. The prefix is `plugin.` plus the own ID with `.` as
`_`. A hook for themes to aim at; keeping it compatible is the author's
call.

### `CreatableNodeId`

```ts
type CreatableNodeId = CoreCreatableNodeId | PluginNodeId;
```

IDs a plugin may create. `app.*` / `chrome.*` / `nav.*` / `chat.*` cannot
be created (tied to real domain objects).

### `CoreCreatableNodeId`

Creatable core-side union: `overlay.*`, `settings.*`, `primitive.*`,
`layout.*` (44 total).

### `NodeState`

```ts
type NodeState =
  | "hover" | "active" | "focus" | "selected" | "disabled"
  | "unread" | "mentioned" | "loading" | "grouped" | "collapsed";
```

The same set themes can condition on.

### `DataByNode`

Table from ID to data type. What types `ctx.data`.

```ts
interface DataByNode {
  "nav.guild_list.item": GuildData;
  "chat.message": MessageData;
  /* ... */
}
```

IDs without a mapping read `ctx.data` as `undefined`.

## Events

There is no event subscription API. React with patches:

- Offscreen nodes are never visited, so nothing walks every message.
- Branch with `ui.exists`, transform in the matching patch.
- Keep patches pure; no networking, saving, or counting inside.

Gateway event middleware is a future mechanism; this section grows when
it lands.
