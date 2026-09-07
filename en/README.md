# Gumicord API docs

Author reference for themes and plugins. Every screen part carries a
**stable ID** (names like `chat.message.content`); themes name IDs to
decide looks, plugins name IDs to register transforms. IDs with roles
live in the [stable ID catalog](en/theme/ids.md).

## Which to read

- To change looks → [Theme API](en/theme.md). JSON is enough to build
  one. To build the first one, start at
  [Setup and authoring](en/theme/setup.md).
- To add screen parts → [Plugin API](en/plugins.md). For readers of
  TypeScript. To build the first one, start at
  [Setup and authoring](en/plugins/setup.md).
- To browse signatures → [Plugin API reference](en/plugins/reference.md).

## Contents

- [Theme API](en/theme.md) — overview, minimal example, child guide
  - [Setup and authoring](en/theme/setup.md) — tooling, first steps, samples
  - [Stable ID catalog](en/theme/ids.md) — IDs writable in `select`, with roles
  - [Manifest and tokens](en/theme/manifest-tokens.md) — identity, value table, `$data.tint`
  - [Rules and conditions](en/theme/rules.md) — `select`, `when`, cascade, selection
  - [Styles](en/theme/styles.md) — properties, `transition`, inline slots
  - [Backgrounds, assets, breakage](en/theme/assets.md) — images, reference forms, validation
- [Plugin API](en/plugins.md) — overview, minimal example, child guide
  - [API reference](en/plugins/reference.md) — signatures of functions, `ctx`, interfaces, types, events
  - [Setup and authoring](en/plugins/setup.md) — tooling, first steps, samples
  - [Execution model and lifetime](en/plugins/lifecycle.md) — isolation, chaining rules, budgets, failure counting
  - [Manifest, capabilities, approval](en/plugins/manifest.md) — `manifest.json`, capabilities, approval and states, settings
  - [Building, distribution, don'ts](en/plugins/build.md) — workflow, prohibitions
