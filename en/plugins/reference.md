# Plugin API reference

A signature-browsing page. For flows see the concept pages:
[Execution model and lifetime](en/plugins/lifecycle.md),
[Manifest, capabilities, approval](en/plugins/manifest.md),
[Building, distribution, don'ts](en/plugins/build.md).
To build the first one, start at [Setup and authoring](en/plugins/setup.md).

- There is no event subscription API. See [Events](#events).

## Contents

- [`ctx`](#ctx): the context a patch receives
- [`ui`](#ui): `patch`, `exists`, `wrap`, `after`, `before`, `settings`, `stack`, `node`, `text`, `badge`, `button`, `icon`
- [`log`](#log): `info`, `warn`, `error`
- [`storage`](#storage): `get`, `set`, `remove`, `getJSON`, `setJSON`
- [Interfaces](#interfaces): `UINode`, `NewUINode`, `PatchContext`, `PatchFn`, 8 data types
- [Type aliases and enum-like types](#type-aliases-and-enum-like-types): `NodeId`, `PluginNodeId`, `CreatableNodeId`, `CoreCreatableNodeId`, `NodeState`, `DataByNode`
- [Events](#events): no subscription API

## `ctx`

The context a patch receives. Passed as `{ data }`, taken out per patch
for that node alone.

- `ctx.data` holds per-node domain facts, typed from the ID
  (`ctx.data.author.bot`); `undefined` where the node carries none.
- Read-only; no raw payloads ever surface. Additions are non-breaking;
  removals and renames are breaking (part of the ABI).
- Which ID carries which type lives in the `data` column of the
  [stable ID catalog](en/theme/ids.md). The type list sits under
  [Interfaces](#interfaces).

```ts
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;  // branch on data
  return ui.after(node, ui.badge({ text: "BOT" }));
});
```

<!-- BEGIN GENERATED: api -->

## `ui`

### `ui.patch`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
```

Registers a node transform against a stable ID.

How it applies:
- traversal is bottom-up, children before their parent
- matching uses the stable ID as it was before any patch
- a patch's output is not recursed into; it is final
- several patches on one node chain in registration order

So a patch runs exactly once per node.

Registering against a node that does not exist here (`chrome.*` on
mobile) is not an error; it simply never runs. To branch beforehand,
use `exists`.
Virtualisation means offscreen nodes are never visited, so
nothing can walk every message. Use Gateway event middleware instead.

`fn` must be pure: how many times it runs for one message is
not defined, since it runs again each time the node leaves the screen
and comes back, and a side effect would not add up.

`ctx.data` is typed from `id`, so registering against
`chat.message.header.author` types `ctx.data.author.bot`.

```ts
ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

### `ui.exists`

```ts
exists(id: NodeId): boolean
```

Whether the node can exist in this environment.

`chrome.*` does not exist on mobile. Patching a missing ID is harmless,
so this is only for branching beforehand.

### `ui.wrap`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

Wraps a node as the child of another.

`wrapper` cannot be a core ID such as `chat.*`: a plugin transforms the
nodes it is given, it does not manufacture core ones. Only its own
namespace and `primitive.*` / `layout.*` may wrap.

### `ui.after`

```ts
after(node: UINode, sibling: UINode): UINode
```

Adds a sibling after the node.

### `ui.before`

```ts
before(node: UINode, sibling: UINode): UINode
```

Adds a sibling before the node.

### `ui.settings`

```ts
settings(fn: () => UINode): void
```

Provides this plugin's settings page, shown in the client's settings
screen when the manifest declares a `settings` entry.

Display-only for now: controls sit inert until the settings event
channel arrives, so describe, do not operate.

### `ui.stack`

```ts
stack(nodes: UINode[]): UINode
```

Stacks nodes vertically.

### `ui.node`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

Creates any creatable node, for a plugin's own IDs.

### `ui.text`

```ts
text(value: string): NewUINode
```

Creates a `primitive.text` node holding the string.

### `ui.badge`

```ts
badge(opts: { text: string; tone?: string }): NewUINode
```

Creates a `primitive.badge` node.

`tone` is theme-owned and drops; the badge keeps its text.

### `ui.button`

```ts
button(opts: { label: string; onPress: () => void }): NewUINode
```

Creates a `primitive.button` node.

Functions cannot cross the host boundary, so `onPress` drops silently
and the button is inert on settings screens.

### `ui.icon`

```ts
icon(name: string): NewUINode
```

Creates a `primitive.icon` node showing the named picture.

## `log`

### `log.info`

```ts
info(msg: string): void
```

Records an informational message.

### `log.warn`

```ts
warn(msg: string): void
```

Records a warning.

### `log.error`

```ts
error(msg: string): void
```

Records an error.

## `storage`

### `storage.get`

```ts
get(key: string): string | null
```

Reads a key. Yields `null` when missing.

### `storage.set`

```ts
set(key: string, value: string): void
```

Writes a key. Persists immediately; never call from inside a patch.

### `storage.remove`

```ts
remove(key: string): void
```

Deletes a key. Persists immediately; never call from inside a patch.

### `storage.getJSON`

```ts
getJSON<T>(key: string, fallback: T): T
```

Reads JSON. Returns `fallback` for both missing and broken JSON.

Never call from inside a patch.

### `storage.setJSON`

```ts
setJSON(key: string, value: unknown): void
```

Writes JSON-encoded. Never call from inside a patch.

## Interfaces

### `UINode`

```ts
interface UINode {
  /** The stable ID. */
  id: NodeId | PluginNodeId;
  /** Distinguishes siblings sharing an id under one parent. Read-only. */
  readonly key?: string;
  /** The states currently held. */
  readonly states?: readonly NodeState[];
  /**
   * The colour the data carries (`#RRGGBB`): a role colour, a folder colour.
   *
   * Not a style. Where it lands is the theme's choice, and it only fills a
   * property written as `$data.tint`.
   */
  readonly tint?: string;
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

### `NewUINode`

```ts
interface NewUINode extends UINode {
  id: CreatableNodeId;
}
```

A node a plugin creates. Core IDs are not allowed.

### `PatchContext`

```ts
interface PatchContext<Id extends NodeId = NodeId> {
  readonly data: Id extends keyof DataByNode ? Readonly<DataByNode[Id]> : undefined;
}
```

The context a patch receives.

`data` is typed from the stable ID, so `ctx.data.author.bot` is type
safe. It is `undefined` on a node that carries none.

### `PatchFn`

```ts
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

A node transform.

It must be pure. Virtualisation leaves it undefined how many
times it runs for one message — again each time the node leaves the
screen and comes back — so a side effect is unpredictable. To react to
something happening, use Gateway event middleware.

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

The domain objects exposed as `data`.

These are part of the extension ABI too: adding is not breaking, but
removing and renaming are.

Discord's raw payloads are never exposed. Exposing one would make it
part of the ABI and tie us to Discord's changes.

Which node carries which type is set by `DataByNode` in `ids.ts`.

One Discord user, as a plugin may see them.

Fields: `id`・`username`・`displayName`・`bot`・`avatarUrl?`

### `MessageData`

```ts
interface MessageData {
  readonly id: string;
  readonly channelId: string;
  readonly guildId?: string;
  readonly createdAt: string;
  readonly editedAt?: string;
  /** Plain text. Parsed Markdown appears as nodes. */
  readonly content: string;
  readonly author: UserData;
  readonly pinned: boolean;
  readonly referencedMessageId?: string;
}
```

One chat message. `content` is plain text; the decorated body lives in nodes.

Fields: `id`・`channelId`・`guildId?`・`createdAt`・`editedAt?`・`content`・`author`・`pinned`・`referencedMessageId?`

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

One guild, with its unread state.

Fields: `id`・`name`・`iconUrl?`・`unread`・`mentionCount`

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

One channel, with its unread state.

Fields: `id`・`name`・`type`・`topic?`・`nsfw`・`unread`・`mentionCount`

### `CategoryData`

```ts
interface CategoryData {
  readonly id: string;
  readonly name: string;
  readonly collapsed: boolean;
}
```

One channel category.

Fields: `id`・`name`・`collapsed`

### `DmData`

```ts
interface DmData {
  readonly id: string;
  readonly recipients: readonly UserData[];
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

One direct message thread.

Fields: `id`・`recipients`・`unread`・`mentionCount`

### `MemberData`

```ts
interface MemberData {
  readonly user: UserData;
  /** Their name in this guild, or `user.displayName` if unset. */
  readonly displayName: string;
  /** `online` / `idle` / `dnd` / `offline` */
  readonly status: string;
  readonly roles: readonly string[];
}
```

One person in the member list.

Roles appear by name, not identifier: an identifier means nothing to a
user, and a plugin showing one would just show a number.

Fields: `user`・`displayName`・`status`・`roles`

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

One message attachment.

Fields: `id`・`filename`・`size`・`contentType?`・`url`・`width?`・`height?`

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

One message embed.

Fields: `type`・`title?`・`description?`・`url?`・`color?`

## Type aliases and enum-like types

### `NodeId`

```ts
type NodeId =
  | "app.root"
  | "app.window"
  | /* ... 118 more */
  | "layout.scrollbar.thumb"
  ;
```

A UITree stable ID. An unknown one fails to build.

For the full list see the [stable ID catalog](en/theme/ids.md).

### `PluginNodeId`

```ts
type PluginNodeId = `plugin.${string}`;
```

An ID a plugin creates in its own namespace.

The prefix is `plugin.` followed by the plugin ID with `.` replaced by
`_`. It is a hook for themes to aim at, and its compatibility is the
plugin author's to keep.

### `CreatableNodeId`

```ts
type CreatableNodeId = CoreCreatableNodeId | PluginNodeId;
```

The IDs a plugin may create.

Not `app.*`, `chrome.*`, `nav.*` or `chat.*`: those are tied to real
domain objects, and forging one would make the accessibility tree lie
and let another plugin's selector match a node that is not there.

A plugin transforms the nodes it is given; it does not manufacture core
ones.

### `CoreCreatableNodeId`

```ts
type CoreCreatableNodeId =
  | "overlay.layer"
  | "overlay.scrim"
  | /* ... 41 more */
  | "layout.scrollbar.thumb"
  ;
```

The IDs a plugin may create.

A core node is tied to a real domain object, so a plugin able to forge
one would make the accessibility tree lie.
See spec/03-uitree.md 8.2.

### `NodeState`

```ts
type NodeState =
  | "hover"
  | "active"
  | "focus"
  | "selected"
  | "disabled"
  | "unread"
  | "mentioned"
  | "loading"
  | "grouped"
  | "collapsed";
```

A node state, matching what a theme can condition on.

### `DataByNode`

```ts
interface DataByNode {
  "nav.guild_list.item": GuildData;
  "nav.guild_list.item.icon": GuildData;
  "nav.guild_list.item.pill": GuildData;
  "nav.guild_list.item.badge": GuildData;
  "nav.channel_list.category": CategoryData;
  "nav.channel_list.item": ChannelData;
  "nav.channel_list.item.icon": ChannelData;
  "nav.channel_list.item.name": ChannelData;
  "nav.channel_list.item.badge": ChannelData;
  "nav.dm_list.item": DmData;
  "nav.member_list.item": MemberData;
  "nav.member_list.item.avatar": MemberData;
  "nav.member_list.item.presence": MemberData;
  "nav.member_list.item.name": MemberData;
  "chat.header": ChannelData;
  "chat.header.title": ChannelData;
  "chat.header.topic": ChannelData;
  "chat.message": MessageData;
  "chat.message.avatar": MessageData;
  "chat.message.header": MessageData;
  "chat.message.header.author": MessageData;
  "chat.message.header.badges": MessageData;
  "chat.message.header.timestamp": MessageData;
  "chat.message.reply_ref": MessageData;
  "chat.message.content": MessageData;
  "chat.message.attachments": MessageData;
  "chat.message.attachment": AttachmentData;
  "chat.message.embeds": MessageData;
  "chat.message.embed": EmbedData;
  "chat.message.actions": MessageData;
}
```

The `data` each node kind carries.

<!-- END GENERATED: api -->

## Events

There is no event subscription API. React with patches:

- Offscreen nodes are never visited, so nothing walks every message.
- Branch with `ui.exists`, transform in the matching patch.
- Keep patches pure; no networking, saving, or counting inside.

Gateway event middleware is a future mechanism; this section grows when
it lands.
