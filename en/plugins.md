# Plugin API

## Getting started

Plugins are TypeScript running in an isolated scripting environment
inside the client. They do one
thing: **register transforms on nodes with stable IDs**. Redrawing, state,
and persistence belong to the host; plugins never touch them.

To get running:

1. Put `manifest.json` and `src/index.ts` in an empty directory (below).
2. Bundle with `npx gumicord-plugin build <dir>` into `plugin.js`.
3. Drop the directory into the plugins folder and start the client
   (first-seen permissions ask approval).

Minimal example (`src/index.ts`):

```ts
import { log, ui } from "@gumicord/sdk";

log.info("hello plugin loaded");

ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

## Execution model

- One isolated runtime pair per plugin. No cross references.
- Patch chain (P1–P7):
  - P1: bottom-up traversal (children before parents).
  - P2: matching uses the pre-apply stable ID.
  - P3: no recursing into output. Output is final.
  - P4: same-node patches chain in registration order.
  - P5: exceptions revert their node. Nothing else is touched.
  - P6: plugins chain in load order. Each sees the previous output.
  - P7: patches are pure functions. No side effects.
- Budgets: infinite loops die at 100ms. Chains get 8ms (previous output
  plus warning past it). Memory cap 32MB, stack 512KB.
- Failures revert their branch and count under the plugin ID. 100 per 60
  seconds disables the plugin with notice. Re-enable from settings.
- Non-`0` `GUMICORD_SAFE_MODE` starts without loading any plugin.

## `manifest.json`

No unknown top-level keys.

| Field | Required | Shape |
|---|---|---|
| `id` | required | Reverse domain (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`). Matches the directory name |
| `name` | required | 1–64 chars. User-visible name |
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

## `ui`

### `ui.patch(id, fn)`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

Registers a transform on a stable ID. Effects:

- Bottom-up traversal; children arrive already patched.
- Matching uses the pre-apply ID. Output is not recursed into.
- Same-node patches chain in registration order.
- Registering on a missing ID is harmless (never runs). Branch beforehand
  with `ui.exists`.
- Virtualized offscreen nodes are never visited. React to events via
  Gateway event middleware instead.
- Keep `fn` pure. Calls per message are undefined (re-runs each time the
  node leaves the screen and returns), so counters or fetches inside
  misbehave.

Arguments:

- `id`: stable ID (`NodeId`). Type-checked; unknown IDs do not pass.
- `fn`: `(node, ctx) => UINode`. The returned node is used as-is.
- `ctx.data`: below. Typed from the ID (`ctx.data.author.bot`); `undefined`
  where the node carries none.

### `ui.exists(id)`

```ts
exists(id: NodeId): boolean
```

Whether the ID can exist here (`chrome.*` is absent on mobile).
Registering on a missing ID is already harmless, so this is only for
beforehand branching.

- Argument: `id` (stable ID).
- Returns: `true` when it can exist.

### `ui.wrap(node, wrapper)`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

Wraps a node as a child. Effect: the original lands under
`children: [node]` of a new parent.

- Arguments: wrapped node, parent (no `children` passed).
- Returns: the wrapped node.
- Limit: core IDs (`app.*` / `chrome.*` / `nav.*` / `chat.*`) cannot wrap.
  Own namespace plus `primitive.*` / `layout.*` only.

### `ui.after(node, sibling)` / `ui.before(node, sibling)`

```ts
after(node: UINode, sibling: UINode): UINode
before(node: UINode, sibling: UINode): UINode
```

Adds a sibling after / before. Effect: wraps in
`{ id: "layout.row", children: [...] }`. Output counts as final.

- Arguments: target node and sibling.
- Returns: the wrapping row node.

### `ui.stack(nodes)`

```ts
stack(nodes: UINode[]): UINode
```

Stacks vertically. Effect: `{ id: "layout.column", children: nodes }`.

- Argument: nodes to stack.
- Returns: the column node.

### `ui.node(id, props?, children?)`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

Creates an own-namespace node. Effect: returns `{ id, props?, children? }`.

- Arguments: `id` (creatable IDs only: `plugin.` + own ID with `.` as `_`,
  or `primitive.*` / `layout.*`), `props`, `children`.
- Returns: the new node.
- Limit: forging a core ID discards that whole output.

### `ui.text(value)` / `ui.badge(opts)` / `ui.button(opts)` / `ui.icon(name)`

```ts
text(value: string): NewUINode
badge(opts: { text: string; tone?: string }): NewUINode
button(opts: { label: string; onPress: () => void }): NewUINode
icon(name: string): NewUINode
```

Create `primitive.text` / `primitive.badge` / `primitive.button` /
`primitive.icon`. Effect: a node with that `id` and `props`.

- Arguments: display string, badge mark, label plus press action, picture name.
- Returns: the new node.
- Note: functions (`onPress`) cannot cross the host boundary and drop
  silently. Theme-owned specs (`tone` etc.) drop too. `button` is inert
  on settings screens.

### `ui.settings(fn)`

```ts
settings(fn: () => UINode): void
```

Registers a display-only settings page. Effect: the page shows in settings.

- Argument: node factory, called at display time.
- Returns: nothing.
- Limit: no actionable widgets yet (their event channel arrives
  separately). Persist from the settings side, never from a patch.

## `log`

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

`log` capability logging. Effect: lands in the host log.

- Argument: message.
- Returns: nothing.

## `storage`

Small host-side store. Split per plugin, survives reloads.

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

## `ctx.data`

Per-patch context `{ data }`. Per-node domain facts typed from the ID
(`ctx.data.author.bot`); `undefined` where the node carries none.
Read-only; no raw payloads ever surface. Additions are non-breaking;
removals and renames are breaking (part of the ABI).

| Type | Main fields |
|---|---|
| `UserData` | `id`・`username`・`displayName`・`bot`・`avatarUrl?` |
| `MessageData` | `id`・`channelId`・`guildId?`・`createdAt`・`editedAt?`・`content` (plain; decorated body lives in nodes)・`author`・`pinned`・`referencedMessageId?` |
| `GuildData` | `id`・`name`・`iconUrl?`・`unread`・`mentionCount` |
| `ChannelData` | `id`・`name`・`type`・`topic?`・`nsfw`・`unread`・`mentionCount` |
| `CategoryData` | `id`・`name`・`collapsed` |
| `DmData` | `id`・`recipients`・`unread`・`mentionCount` |
| `MemberData` | `user`・`displayName`・`status` (`online`/`idle`/`dnd`/`offline`)・`roles` (names, not IDs) |
| `AttachmentData` | `id`・`filename`・`size`・`contentType?`・`url`・`width?`・`height?` |
| `EmbedData` | `type`・`title?`・`description?`・`url?`・`color?` |

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

`NodeState` is `hover`・`active`・`focus`・`selected`・`disabled`・
`unread`・`mentioned`・`loading`・`grouped`・`collapsed`.

## Host boundary

Only structure and draw content cross. `key`, states, colors, and
references inherit from the input tree; the JS side is never trusted.

- Incoming `content` per kind: `text` string, `icon` name, `image` URL,
  `qr` value, `rich` runs (`text` plus `font`, `color {r,g,b,a}`,
  `under`, `through`, `hidden`, `revealed`, `link`, `image`), `editable`
  field (`text`, `caret`, `selection {start,end}`, `composing`,
  `placeholder`).
- `props` backfills only when `content` is absent, from its first
  `value`/`text`/`label` or `name`/`url`.

## Settings screens

Render into `settings.*`. No actionable widgets yet. Persist from the
settings side.

## Approval and states

First-seen permissions wait for the user. Stored as
`{"grants": {id: [...]}, "disabled": [id...]}` (`grants.json`).
Empty means denied, missing key means unconfirmed (different things).
Disabling stops while keeping the grant; re-enabling does not re-ask.
Denials and revokes live in settings.

## Building and distribution

- Development language is TypeScript + `@gumicord/sdk` (`.d.ts` checked).
- `gumicord-plugin dev` watches with esbuild plus hot reload.
- `gumicord-plugin build` minifies to `plugin.js` (plus optional qjsc).
- Distribute one directory per plugin.

## Don't

- Identify operations by display strings.
- Do heavy work, networking, or saving inside patches (killed at 100ms).
- Hand-edit `plugin.js` or hand-widen `manifest.json`.
- Bring side effects into same-node patches (P7; virtualization calls
  unpredictably often).
