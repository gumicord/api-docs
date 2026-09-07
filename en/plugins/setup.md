# Setup and authoring

## Tooling

- Node.js and TypeScript. The development language is TypeScript plus
  `@gumicord/sdk` (type checking and completion).
- Pick an editor with type completion. Unknown stable IDs do not pass the
  type check, so typos surface while writing.

## Layout

Put `manifest.json` (identity) and `src/index.ts` (transforms) in an empty
directory. Distribute one directory per plugin.

```
myplugin/
├─ manifest.json
└─ src/
   └─ index.ts
```

Minimal `manifest.json`:

```json
{
  "$schema": "https://gumicord.dev/schema/plugin-manifest-1.json",
  "id": "dev.example.myplugin",
  "name": "My plugin",
  "version": "1.0.0",
  "capabilities": ["log"]
}
```

For the fields see [Manifest, capabilities, approval](en/plugins/manifest.md).

## First steps

1. Place the two files above (copy the sample below).
2. Bundle with `npx gumicord-plugin build <dir>` into `plugin.js`.
3. Drop the directory into the plugins folder and start the client.
4. First-seen permissions ask approval. Until granted, that permission
   counts as absent.
5. Grants and denials persist.

While developing, use `gumicord-plugin dev` (file watch plus hot
reload).

## Authoring

The basic shape is just "name an ID, register a function":

```ts
import { ui } from "@gumicord/sdk";

ui.patch("chat.message.header.author", (node) => node);
```

Branch on [ctx](en/plugins/reference.md#ctx):

```ts
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;  // touch BOT rows only
  return ui.after(node, ui.badge({ text: "BOT" }));
});
```

Keep:

- Patches pure. Call counts are undefined, so counters, fetches, and
  other side effects misbehave.
- No heavy work, networking, or saving inside patches (killed at 100ms).
- Registering on a missing ID is harmless. Branch beforehand with
  `ui.exists`.

## Sample

Full example (`src/index.ts`; read line by line):

```ts
import { log, ui } from "@gumicord/sdk";

log.info("hello plugin loaded");  // lands in the log. Startup check

// Badge next to BOT author names
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;
  return ui.after(node, ui.badge({ text: "BOT" }));
});

// Count badge on channels with unread mentions
ui.patch("nav.channel_list.item", (node, ctx) => {
  if (ctx.data.mentionCount === 0) return node;
  return ui.after(node, ui.badge({ text: String(ctx.data.mentionCount) }));
});
```

What the first patch changes: BOT rows gain a badge next to the author
name. Human rows stay untouched. What the second patch changes: channels
with unread mentions gain a count. Rows without unread stay untouched.

Read next: [API reference](en/plugins/reference.md),
[Execution model and lifetime](en/plugins/lifecycle.md).
