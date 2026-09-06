# プラグイン API リファレンス

署名だけを眺めるための頁。使い方の流れは概念頁を見ること:
[実行モデルと寿命](ja/plugins/lifecycle.md)・
[manifest と能力・承認](ja/plugins/manifest.md)・
[`ui`](ja/plugins/ui.md)・
[`ctx.data` とノードの形](ja/plugins/data.md)。

- クラスは無い。`ui`・`log`・`storage` は名前空間であり、`new` しない。
- `enum` キーワードは無い。`NodeId`・`NodeState` 等は文字列リテラルの
  合併型であり、列挙のように使う。
- イベント購読 API は無い。詳しくは [イベント](#イベント) を見ること。

## 目次

- [`ui`](#ui): `patch`・`exists`・`wrap`・`after`・`before`・`settings`・`stack`・`node`・`text`・`badge`・`button`・`icon`
- [`log`](#log): `info`・`warn`・`error`
- [`storage`](#storage): `get`・`set`・`remove`・`getJSON`・`setJSON`
- [インターフェース](#インターフェース): `UINode`・`NewUINode`・`PatchContext`・`PatchFn`・データ8種
- [型エイリアスと列挙的型](#型エイリアスと列挙的型): `NodeId`・`PluginNodeId`・`CreatableNodeId`・`CoreCreatableNodeId`・`NodeState`・`DataByNode`
- [イベント](#イベント): 購読 API なし

## `ui`

### `ui.patch`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
```

安定 ID に変形を登録する。

- 引数: `id` (安定 ID。型で検査され、未知 ID は通らない)、`fn` (`(node, ctx) => UINode`。返したノードがそのまま使われる)。
- 返す値: なし。
- 効果: 走査はボトムアップで、子は適用済みの状態で渡ってくる。照合は適用前の ID に対して行い、出力には再帰しない。同一ノードの複数パッチは登録順に連鎖する。
- 注意: 存在しない ID への登録は無害 (走らないだけ)。仮想化で画面外のノードは訪れない。`fn` は純粋に書く (呼ばれる回数は不定)。

```ts
ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

### `ui.exists`

```ts
exists(id: NodeId): boolean
```

- 引数: `id` (安定 ID)。
- 返す値: その ID がこの環境に存在しうれば `true` (`chrome.*` はモバイルに無い)。
- 効果: 存在確認のみ。登録自体は存在しない ID でも無害なので、事前分岐のためだけに使う。

### `ui.wrap`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

- 引数: `node` (包まれる側)、`wrapper` (親。`children` は渡さない)。
- 返す値: 元ノードを `children: [node]` の新しい親の下に置いたノード。
- 制限: 包みに core ID (`app.*` / `chrome.*` / `nav.*` / `chat.*`) は使えない。自名前空間と `primitive.*` / `layout.*` のみ。

### `ui.after`

```ts
after(node: UINode, sibling: UINode): UINode
```

- 引数: 対象ノードと兄弟ノード。
- 返す値: `{ id: "layout.row", children: [node, sibling] }` で包んだ行ノード。
- 効果: 後に兄弟を足す。出力は最終形として扱われる。

### `ui.before`

```ts
before(node: UINode, sibling: UINode): UINode
```

- 引数: 対象ノードと兄弟ノード。
- 返す値: `{ id: "layout.row", children: [sibling, node] }` で包んだ行ノード。
- 効果: 前に兄弟を足す。出力は最終形として扱われる。

### `ui.settings`

```ts
settings(fn: () => UINode): void
```

- 引数: ノード工場。呼ばれるのは表示時。
- 返す値: なし。
- 効果: 表示専用の設定頁を登録する。設定画面にその頁が出る。
- 制限: 操作部品は置いても押せない。保存は設定画面の側で行い、パッチの中から `storage.set` を呼ばない。

### `ui.stack`

```ts
stack(nodes: UINode[]): UINode
```

- 引数: 積むノード列。
- 返す値: `{ id: "layout.column", children: nodes }` の縦列ノード。

### `ui.node`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

- 引数: `id` (作れる ID のみ。`plugin.`＋自 ID の `.`→`_`、または `primitive.*` / `layout.*`)、`props` (任意の付随値)、`children`。
- 返す値: `{ id, props?, children? }` の新ノード。
- 制限: core ID (`app.*` 等) を作るとその出力ごと捨てられる。

### `ui.text`

```ts
text(value: string): NewUINode
```

- 引数: 表示文字列。
- 返す値: `primitive.text` ノード (`props.value` に文字列を持つ)。

### `ui.badge`

```ts
badge(opts: { text: string; tone?: string }): NewUINode
```

- 引数: 標識 (`text` と任意の `tone`)。
- 返す値: `primitive.badge` ノード。
- 注意: `tone` 等の見た目指定はテーマの持ち物のため落とす。

### `ui.button`

```ts
button(opts: { label: string; onPress: () => void }): NewUINode
```

- 引数: 標識＋押下動作。
- 返す値: `primitive.button` ノード。
- 注意: `onPress` 等の関数はホスト境界を越えられず黙って落とす。`button` は設定画面では押せない。

### `ui.icon`

```ts
icon(name: string): NewUINode
```

- 引数: 絵文字名。
- 返す値: `primitive.icon` ノード (`props.name` に名を持つ)。

## `log`

`log` 能力の記録。効果: 記録に残る。引数は記録文。返す値なし。

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

## `storage`

ホスト側の小さな置き場。プラグイン毎に分かれ、再読込を越えて生きる。

### `storage.get`

```ts
get: (key: string) => string | null
```

- 引数: 鍵。
- 返す値: 不在で `null`。

### `storage.set`

```ts
set: (key: string, value: string) => void
```

- 引数: 鍵と値。
- 効果: 即時保存される。
- 制限: パッチの中から呼ばない (設定画面の側で保存する)。

### `storage.remove`

```ts
remove: (key: string) => void
```

- 引数: 鍵。
- 効果: 即時保存される。
- 制限: パッチの中から呼ばない。

### `storage.getJSON`

```ts
getJSON<T>(key: string, fallback: T): T
```

- 引数: 鍵と `fallback` (不在時と壊れJSONの両方で返る)。
- 返す値: 保存値、または `fallback`。

### `storage.setJSON`

```ts
setJSON(key: string, value: unknown): void
```

- 引数: 鍵と値 (JSON 化して保存する)。
- 制限: パッチの中から呼ばない。

## インターフェース

### `UINode`

```ts
interface UINode {
  id: NodeId | PluginNodeId;
  readonly key?: string;
  readonly states?: readonly NodeState[];
  readonly tint?: string;
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

- `id`: 安定 ID。
- `key`: 同親下の同 ID 兄弟の区別。読取専用。
- `states`: 現在の状態。読取専用。
- `tint`: データ由来色 (`#RRGGBB`)。書く場所はテーマが決める。読取専用。
- `props` / `children`: 付随値と子。

### `NewUINode`

```ts
interface NewUINode extends UINode {
  id: CreatableNodeId;
}
```

作る側のノード。`id` が作れる ID に限られる以外は `UINode` と同じ。
`key`・`states`・色・参照は入力木から継承され、出力側で変えられない。

### `PatchContext`

```ts
interface PatchContext<Id extends NodeId = NodeId> {
  readonly data: Id extends keyof DataByNode ? Readonly<DataByNode[Id]> : undefined;
}
```

パッチが受け取る文脈。`data` は ID から型が付き (`ctx.data.author.bot` 等)、
該当なしは `undefined`。読取専用。`Context` は別名である。

### `PatchFn`

```ts
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

ノード変形。純粋関数でなければならない (呼ばれる回数は不定のため)。

### `UserData`

```ts
interface UserData {
  readonly id: string;
  readonly username: string;
  readonly displayName: string;
  readonly bot: boolean;
  readonly avatarUrl?: string;
}
```

### `MessageData`

```ts
interface MessageData {
  readonly id: string;
  readonly channelId: string;
  readonly guildId?: string;
  readonly createdAt: string;
  readonly editedAt?: string;
  readonly content: string;
  readonly author: UserData;
  readonly pinned: boolean;
  readonly referencedMessageId?: string;
}
```

`content` は素文。装飾済み本文はノード側にある。

### `GuildData`

```ts
interface GuildData {
  readonly id: string;
  readonly name: string;
  readonly iconUrl?: string;
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `ChannelData`

```ts
interface ChannelData {
  readonly id: string;
  readonly name: string;
  readonly type: string;
  readonly topic?: string;
  readonly nsfw: boolean;
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `CategoryData`

```ts
interface CategoryData {
  readonly id: string;
  readonly name: string;
  readonly collapsed: boolean;
}
```

### `DmData`

```ts
interface DmData {
  readonly id: string;
  readonly recipients: readonly UserData[];
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

### `MemberData`

```ts
interface MemberData {
  readonly user: UserData;
  readonly displayName: string;
  readonly status: string;
  readonly roles: readonly string[];
}
```

`status` は `online` / `idle` / `dnd` / `offline`。`roles` は ID ではなく名列。

### `AttachmentData`

```ts
interface AttachmentData {
  readonly id: string;
  readonly filename: string;
  readonly size: number;
  readonly contentType?: string;
  readonly url: string;
  readonly width?: number;
  readonly height?: number;
}
```

### `EmbedData`

```ts
interface EmbedData {
  readonly type: string;
  readonly title?: string;
  readonly description?: string;
  readonly url?: string;
  readonly color?: number;
}
```

## 型エイリアスと列挙的型

### `NodeId`

安定 ID の合併型 (121 個。未知 ID は型で通らない)。一覧は
[安定 ID カタログ](ja/theme/ids.md) を見ること。

```ts
type NodeId = "app.root" | "app.window" | /* ... */ | "layout.scrollbar.thumb";
```

### `PluginNodeId`

```ts
type PluginNodeId = `plugin.${string}`;
```

自名前空間の ID。接頭辞は `plugin.`＋自 ID の `.`→`_`。テーマが狙う
ための掛け鉤であり、互換性の維持は作者の責任である。

### `CreatableNodeId`

```ts
type CreatableNodeId = CoreCreatableNodeId | PluginNodeId;
```

プラグインが作れる ID。`app.*` / `chrome.*` / `nav.*` / `chat.*` は
作れない (実在の対象と結びついているため)。

### `CoreCreatableNodeId`

作れる core 側 ID の合併型: `overlay.*`・`settings.*`・`primitive.*`・
`layout.*` (44 個)。

### `NodeState`

```ts
type NodeState =
  | "hover" | "active" | "focus" | "selected" | "disabled"
  | "unread" | "mentioned" | "loading" | "grouped" | "collapsed";
```

テーマが条件にできる状態と同じ集合である。

### `DataByNode`

ID からデータ型への対応表。`ctx.data` の型付けの根拠である。

```ts
interface DataByNode {
  "nav.guild_list.item": GuildData;
  "chat.message": MessageData;
  /* ... */
}
```

対応が無い ID の `ctx.data` は `undefined` である。

## イベント

出来事を購読する API は無い。出来事への反応はパッチで行う:

- 画面外のノードは訪れないため、全件走査はできない。
- `ui.exists` で環境分岐し、該当ノードのパッチで変形する。
- パッチは純粋に保ち、通信・保存・計数は持ち込まない。

Gateway 出来事のミドルウェアは将来の仕組みであり、来たらこの節に足す。
