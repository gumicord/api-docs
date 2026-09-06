# `ui`

## `ui.patch(id, fn)`

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
- `ctx.data`: [data](en/plugins/data.md). Typed from the ID
  (`ctx.data.author.bot`); `undefined` where the node carries none.

## `ui.exists(id)`

```ts
exists(id: NodeId): boolean
```

Whether the ID can exist here (`chrome.*` is absent on mobile).
Registering on a missing ID is already harmless, so this is only for
beforehand branching.

- Argument: `id` (stable ID).
- Returns: `true` when it can exist.

## `ui.wrap(node, wrapper)`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

Wraps a node as a child. Effect: the original lands under
`children: [node]` of a new parent.

- Arguments: wrapped node, parent (no `children` passed).
- Returns: the wrapped node.
- Limit: core IDs (`app.*` / `chrome.*` / `nav.*` / `chat.*`) cannot wrap.
  Own namespace plus `primitive.*` / `layout.*` only.

## `ui.after(node, sibling)` / `ui.before(node, sibling)`

```ts
after(node: UINode, sibling: UINode): UINode
```

Adds a sibling after / before. Effect: wraps in
`{ id: "layout.row", children: [...] }`. Output counts as final.

- Arguments: target node and sibling.
- Returns: the wrapping row node.

## `ui.stack(nodes)`

```ts
stack(nodes: UINode[]): UINode
```

Stacks vertically. Effect: `{ id: "layout.column", children: nodes }`.

- Argument: nodes to stack.
- Returns: the column node.

## `ui.node(id, props?, children?)`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

Creates an own-namespace node. Effect: returns `{ id, props?, children? }`.

- Arguments: `id` (creatable IDs only: `plugin.` + own ID with `.` as `_`,
  or `primitive.*` / `layout.*`), `props`, `children`.
- Returns: the new node.
- Limit: forging a core ID discards that whole output.

## `ui.text(value)` / `ui.badge(opts)` / `ui.button(opts)` / `ui.icon(name)`

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
