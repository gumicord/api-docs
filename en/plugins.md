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

## Contents

- [API reference](en/plugins/reference.md) — signatures of functions, `ctx`, interfaces, types, events
- [Setup and authoring](en/plugins/setup.md) — tooling, first steps, samples
- [Execution model and lifetime](en/plugins/lifecycle.md) — isolation, chaining rules, budgets, failure counting
- [Manifest, capabilities, approval](en/plugins/manifest.md) — `manifest.json`, capabilities, approval and states, settings
- [Building, distribution, don'ts](en/plugins/build.md) — workflow, prohibitions
- [Stable ID catalog](en/theme/ids.md) — IDs with roles (shared with themes)
