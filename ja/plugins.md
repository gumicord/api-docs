# プラグイン API

この文書は TypeScript が読める人向け。Gumicord 自体の知識は仮定しない。

## はじめに

プラグインはクライアントに見た目を足す小さな部品である。まず全体像:

- 画面は**ノードの木** (UITree) である。メッセージ・作者名・ボタン等、
  画面上のあらゆる部品がノードであり、種類ごとに**安定 ID**
  (`chat.message.header.author` のような名前) を持つ。
- プラグインは **ID を指名して変形関数 (パッチ) を登録**する。
  画面が描かれるたび、該当ノードが関数に渡され、返したものが描かれる。
- 使える道具は**宣言した能力だけ**。ホストが注入したもの以外、
  グローバルには何も無い (`fetch` も `fs` も無い)。
- プラグイン同士は互いを知らない。前の出力が次に入るだけである。

動くまでの手順:

1. 空きディレクトリに `manifest.json` (名乗り) と `src/index.ts`
   (変形) を置く (下記)。
2. `npx gumicord-plugin build <dir>` で `plugin.js` を束ねる。
3. そのディレクトリをプラグインフォルダに入れてクライアントを起動する
   (初見の権限は承認窓が出る。許可するまで動かない)。

最小例 (`src/index.ts`):

```ts
import { log, ui } from "@gumicord/sdk";

log.info("hello plugin loaded");

ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

## 目次

- [実行モデルと寿命](ja/plugins/lifecycle.md) — 隔離・P1〜P7・予算・失敗の数え方
- [manifest と能力・承認](ja/plugins/manifest.md) — `manifest.json`・能力・承認と状態・設定画面
- [`ui`](ja/plugins/ui.md) — `patch`・`exists`・`wrap`・`after`/`before`・`stack`・`node`・部品生成
- [`ctx.data` とノードの形](ja/plugins/data.md) — 型表・`UINode`・ホスト境界
- [ビルドと配布・してはいけないこと](ja/plugins/build.md) — 開発手順・禁止事項
- [安定 ID カタログ](ja/theme/ids.md) — ID と役目の一覧 (テーマと共有)
