# Building, distribution, don'ts

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
