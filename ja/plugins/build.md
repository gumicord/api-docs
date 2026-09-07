# ビルドと配布・してはいけないこと

## ビルドと配布

- 開発言語は TypeScript + `@gumicord/sdk` (`.d.ts` で型検査)。
- `gumicord-plugin dev` は esbuild 監視＋ホットリロード。
- `gumicord-plugin build` は esbuild 最小化→`plugin.js` (+任意で qjsc)。
- 1 ディレクトリ 1 プラグインで配る。

## してはいけないこと

- 表示文字列で操作を特定しない (将来の i18n で壊れる)。
- パッチ内で重い処理・通信・保存をしない (100ms で殺される)。
- `plugin.js` の手編集と `manifest.json` の手拡張 (荷下ろし時に弾かれる)。
- 同一ノードへの副作用の持ち込み (パッチは純粋関数。仮想化で呼ばれる回数は不定)。
