# 環境構築と書き方

## 道具

- Node.js と TypeScript。開発言語は TypeScript + `@gumicord/sdk`
  (型検査・補完が効く)。
- エディタは型補完の効くものを選ぶ。未知の安定 ID は型で通らないため、
  打ち間違いは書いている時点で分かる。

## 配置

空きディレクトリに `manifest.json` (名乗り) と `src/index.ts` (変形) を
置く。1 ディレクトリ 1 プラグインで配る。

```
myplugin/
├─ manifest.json
└─ src/
   └─ index.ts
```

`manifest.json` の最小例:

```json
{
  "$schema": "https://gumicord.dev/schema/plugin-manifest-1.json",
  "id": "dev.example.myplugin",
  "name": "My plugin",
  "version": "1.0.0",
  "capabilities": ["log"]
}
```

欄の意味は [manifest と能力・承認](ja/plugins/manifest.md) を見ること。

## 初回の手順

1. 上の 2 ファイルを置く (下のサンプルを写す)。
2. `npx gumicord-plugin build <dir>` で `plugin.js` を束ねる。
3. そのディレクトリをプラグインフォルダに入れてクライアントを起動する。
4. 初見の権限は承認窓が出る。許可するまでその権限は無いものとして動く。
5. 許可・拒否の記録は残る。

開発中は `gumicord-plugin dev` を使う (監視＋ホットリロード)。

## 書き方

基本形は「ID を指名して関数を登録する」だけである:

```ts
import { ui } from "@gumicord/sdk";

ui.patch("chat.message.header.author", (node) => node);
```

条件分岐は `ctx.data` を見る ([ctx](ja/plugins/reference.md#ctx)):

```ts
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;  // BOT の行だけ変える
  return ui.after(node, ui.badge({ text: "BOT" }));
});
```

守ること:

- パッチは純粋に書く。呼ばれる回数は不定のため、カウンタや通信などの
  副作用は予測不能になる。
- 重い処理・通信・保存はパッチの中に書かない (100ms で殺される)。
- 存在しない ID への登録は無害。事前分岐には `ui.exists` を使う。

## サンプル

全文例 (`src/index.ts`。1 行ずつ読む):

```ts
import { log, ui } from "@gumicord/sdk";

log.info("hello plugin loaded");  // 記録に残る。起動の確認用

// BOT の作者名の横に標識を足す
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;
  return ui.after(node, ui.badge({ text: "BOT" }));
});

// 未読があるチャンネルに件数の標識を足す
ui.patch("nav.channel_list.item", (node, ctx) => {
  if (ctx.data.mentionCount === 0) return node;
  return ui.after(node, ui.badge({ text: String(ctx.data.mentionCount) }));
});
```

1 つ目のパッチで変わるもの: BOT の行だけ作者名の横に標識が出る。
人の行は何も変わらない。2 つ目のパッチで変わるもの: 未読のある
チャンネルだけ件数が出る。未読なしの行は何も変わらない。

次に読むもの: [API リファレンス](ja/plugins/reference.md)・
[実行モデルと寿命](ja/plugins/lifecycle.md)。
