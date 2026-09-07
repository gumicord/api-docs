# Building, distribution, don'ts

## Building and distribution

- Development language is TypeScript + `@gumicord/sdk` (type-checked).
- `gumicord-plugin dev` watches files plus hot reload.
- `gumicord-plugin build` bundles and minifies to `plugin.js` (plus optional qjsc).
- Distribute one directory per plugin.

## Don't

- Identify operations by display strings.
- Do heavy work, networking, or saving inside patches (killed at 100ms).
- Hand-edit `plugin.js` or hand-widen `manifest.json`.
- Bring side effects into same-node patches (patches are pure functions;
  virtualization calls unpredictably often).
