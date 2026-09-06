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

## 動きの順序

表から見た一生:

1. フォルダに入れると起動時に見つかり、読み込まれる。
2. 能力付きは初見で承認窓が出る。許可するまでその権限は無いものとして動く。
3. 以降は画面が描かれるたびパッチが走る。重い処理は書けない (100ms で殺される)。
4. 壊れ続けると自動で無効化され、通知される。設定画面から有効化し直せる。
5. 拒否したものは読み込まれない。考え直したら設定画面で承認し直す。

## 実行モデル

- プラグイン毎に独立した実行環境を1組ずつ持つ。他の
  プラグインとは相互参照できない。
- パッチ連鎖 (P1〜P7):
  - P1: 走査はボトムアップ (子→親)。
  - P2: 照合は適用前の安定 ID に対して行う。
  - P3: 出力には再帰しない。出力は最終形。
  - P4: 同一ノードの複数パッチは登録順に連鎖する。
  - P5: 例外のノードは適用前に戻す。他へ波及しない。
  - P6: プラグインは読込順に連鎖する。前の出力が次に入る。
  - P7: パッチは純粋関数。副作用禁止 (呼ばれる回数は不定)。
- 予算: 無限ループは 100ms で殺す。連鎖は 8ms (超えたら前回出力＋警告)。
  メモリ上限 32MB・スタック 512KB。
- 失敗は枝に戻して数え、60 秒に 100 回で無効化し、通知する。
  設定画面から有効化し直せる。
- `GUMICORD_SAFE_MODE` が `0` 以外ならプラグインを読まずに起動する。

## `manifest.json`

余計なトップレベルキーは不可。`$schema` は
`https://gumicord.dev/schema/plugin-manifest-1.json`。

| フィールド | 必須 | 意味・形 |
|---|---|---|
| `id` | 必須 | 逆ドメイン (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`)。収納ディレクトリ名と一致させる |
| `name` | 必須 | 1〜64 文字。表示名 |
| `version` | 必須 | セマンティック版 (`X.Y.Z`、予告可) |
| `entry` | 任意 | 本体名。平らな `.js` / `.qjsc` のみ、省略時は `plugin.js` |
| `description` | 任意 | 512 文字まで。承認画面に出る説明 |
| `capabilities` | 任意 | 重複なし配列。`"log"`・`"storage"` のみ。未知名は読込失敗 |
| `settings` | 任意 | 平らな `.js` / `.qjsc`。省略時は設定画面なし |

## 能力

宣言したものだけが注入される。書いていない API を呼ぶと `TypeError`
になる (拒否ではなく不在として扱う)。

| 能力 | 与えるもの |
|---|---|
| `log` | `log.info` / `warn` / `error` |
| `storage` | `storage.get` / `set` / `remove` |

`ui.exists` と失敗報告は権限ではなく常時注入される。

## `ui`

### `ui.patch(id, fn)`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

安定 ID に変形を登録する。効果:

- 走査はボトムアップで、子は適用済みの状態で渡ってくる。
- 照合は適用前の ID に対して行う。出力には再帰しない。
- 同一ノードの複数パッチは登録順に連鎖する。
- 存在しない ID への登録は無害 (走らないだけ)。事前分岐には
  `ui.exists` を使う。
- 仮想化で画面外のノードは訪れない。出来事への反応は Gateway
  イベントミドルウェアで行う。
- `fn` は純粋に書く。1 メッセージに何度呼ばれるかは不定
  (画面外に出て戻るたびに再実行される) ので、カウンタや通信などの
  副作用は予測不能になる。

引数:

- `id`: 安定 ID (`NodeId`)。型で検査され、未知 ID は通らない。
- `fn`: `(node, ctx) => UINode`。返したノードがそのまま使われる。
- `ctx.data`: 後述。ID から型が付く (`ctx.data.author.bot` 等)。
  該当なしは `undefined`。

### `ui.exists(id)`

```ts
exists(id: NodeId): boolean
```

その ID がこの環境に存在しうるか。`chrome.*` はモバイルに無い。
登録自体は存在しない ID でも無害なので、事前分岐のためだけに使う。

- 引数: `id` (安定 ID)。
- 返す値: 存在しうれば `true`。

### `ui.wrap(node, wrapper)`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

ノードを子に包んで返す。効果: 元ノードを `children: [node]` の
新しい親の下に置く。

- 引数: `node` (包まれる側)、`wrapper` (親。`children` は渡さない)。
- 返す値: 包んだノード。
- 制限: 包みに core ID (`app.*` / `chrome.*` / `nav.*` / `chat.*`)
  は使えない。自名前空間と `primitive.*` / `layout.*` のみ。

### `ui.after(node, sibling)` / `ui.before(node, sibling)`

```ts
after(node: UINode, sibling: UINode): UINode
before(node: UINode, sibling: UINode): UINode
```

後に／前に兄弟を足す。効果: `{ id: "layout.row", children: [...] }`
で包んで返す。出力は最終形として扱われる。

- 引数: 対象ノードと兄弟ノード。
- 返す値: 包んだ行ノード。

### `ui.stack(nodes)`

```ts
stack(nodes: UINode[]): UINode
```

縦に積む。効果: `{ id: "layout.column", children: nodes }` を返す。

- 引数: 積むノード列。
- 返す値: 縦列ノード。

### `ui.node(id, props?, children?)`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

自名前空間のノードを作る。効果: `{ id, props?, children? }` を返す。

- 引数: `id` (作れる ID のみ。`plugin.`＋自 ID の `.`→`_`、または
  `primitive.*` / `layout.*`)、`props` (任意の付随値)、`children`。
- 返す値: 新ノード。
- 制限: core ID (`app.*` 等) を作るとその出力ごと捨てられる。

### `ui.text(value)` / `ui.badge(opts)` / `ui.button(opts)` / `ui.icon(name)`

```ts
text(value: string): NewUINode
badge(opts: { text: string; tone?: string }): NewUINode
button(opts: { label: string; onPress: () => void }): NewUINode
icon(name: string): NewUINode
```

`primitive.text` / `primitive.badge` / `primitive.button` /
`primitive.icon` を作る。効果: 対応する `id` と `props` を持つ
ノードを返す。

- 引数: 表示文字列・標識・標識＋押下動作・絵文字名。
- 返す値: 新ノード。
- 注意: `onPress` 等の関数はホスト境界を越えられず黙って落とす。
  `tone` 等の見た目指定もテーマの持ち物のため落とす。`button` は
  設定画面では押せない。

### `ui.settings(fn)`

```ts
settings(fn: () => UINode): void
```

表示専用の設定頁を登録する。効果: 設定画面にその頁が出る。

- 引数: ノード工場。呼ばれるのは表示時。
- 返す値: なし。
- 制限: 操作部品は置いても押せない (出来事配送が来たら解禁)。
  保存は設定画面の側で行い、パッチの中から `storage.set` を呼ばない。

## `log`

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

`log` 能力の記録。効果: ホストの記録に残る。

- 引数: 記録文。
- 返す値: なし。

## `storage`

ホスト側の小さな置き場。プラグイン毎に分かれ、再読込を越えて生きる。

```ts
get: (key: string) => string | null
set: (key: string, value: string) => void
remove: (key: string) => void
getJSON<T>(key: string, fallback: T): T
setJSON(key: string, value: unknown): void
```

- 引数: 鍵と値。`getJSON` の `fallback` は不在時と壊れJSONの両方で返る。
- 返す値: `get` は不在で `null`。他はなし。
- 効果: `set`/`remove` は即時保存される。
- 制限: パッチの中から呼ばない (設定画面の側で保存する)。

## `ctx.data`

パッチ毎の文脈 `{ data }`。ノード毎のドメイン事実で、ID から型が付く
(`ctx.data.author.bot` 等)。該当なしは `undefined`。読取専用で、
生ペイロードは出ない。追加は非破壊、削除改名は破壊的 (ABI の一部)。

| 型 | 主な欄 |
|---|---|
| `UserData` | `id`・`username`・`displayName`・`bot`・`avatarUrl?` |
| `MessageData` | `id`・`channelId`・`guildId?`・`createdAt`・`editedAt?`・`content` (素文。装飾済み本文はノード側)・`author`・`pinned`・`referencedMessageId?` |
| `GuildData` | `id`・`name`・`iconUrl?`・`unread`・`mentionCount` |
| `ChannelData` | `id`・`name`・`type`・`topic?`・`nsfw`・`unread`・`mentionCount` |
| `CategoryData` | `id`・`name`・`collapsed` |
| `DmData` | `id`・`recipients`・`unread`・`mentionCount` |
| `MemberData` | `user`・`displayName`・`status` (`online`/`idle`/`dnd`/`offline`)・`roles` (ID ではなく名列) |
| `AttachmentData` | `id`・`filename`・`size`・`contentType?`・`url`・`width?`・`height?` |
| `EmbedData` | `type`・`title?`・`description?`・`url?`・`color?` |

## ノードの形 (`UINode`)

```ts
interface UINode {
  id: NodeId | PluginNodeId;  // 安定 ID
  readonly key?: string;      // 同親下の同 ID 兄弟の区別。読取専用
  readonly states?: readonly NodeState[];  // 現在の状態
  readonly tint?: string;     // データ由来色 (#RRGGBB)。書く場所はテーマが決める
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

`NewUINode` は `id` が作れる ID に限られる以外は同じ。`key`・
`states`・色・参照は入力木から継承され、出力側で変えられない
(読むだけなので持ち帰らない)。

`NodeState` は `hover`・`active`・`focus`・`selected`・`disabled`・
`unread`・`mentioned`・`loading`・`grouped`・`collapsed`。

## ホスト境界

渡るのは構造と描画内容のみ。JS 側を信用せず、戻りは節ごとに検証する。

- 入りの `content` は種別毎: `text` 文字列・`icon` 名・`image` URL・
  `qr` 値・`rich` 走り列 (`text`＋`font`・`color {r,g,b,a}`・
  `under`・`through`・`hidden`・`revealed`・`link`・`image`)・
  `editable` 欄 (`text`・`caret`・`selection {start,end}`・
  `composing`・`placeholder`)。
- `content` が無いときだけ `props` の先頭値 (`value`・`text`・`label`
  または `name`・`url`) から補う。

## 設定画面

`settings.*` に出す。操作部品はまだ置けない。保存は設定画面の側で行う。

## 承認と状態

初見の権限は許可されるまで与えない。記録は
`{"grants": {id: [権限...]}, "disabled": [id...]}` (`grants.json`)。
空列は拒否、キー無しは未確認 (別物)。無効化は許可を残して止めるだけ、
有効化は尋ね直さない。拒否のやり直しと取消は設定画面で行う。

## ビルドと配布

- 開発言語は TypeScript + `@gumicord/sdk` (`.d.ts` で型検査)。
- `gumicord-plugin dev` は esbuild 監視＋ホットリロード。
- `gumicord-plugin build` は esbuild 最小化→`plugin.js` (+任意で qjsc)。
- 1 ディレクトリ 1 プラグインで配る。

## してはいけないこと

- 表示文字列で操作を特定しない (将来の i18n で壊れる)。
- パッチ内で重い処理・通信・保存をしない (100ms で殺される)。
- `plugin.js` の手編集と `manifest.json` の手拡張 (荷下ろし時に弾かれる)。
- 同一ノードへの副作用の持ち込み (P7。仮想化で呼ばれる回数は不定)。
