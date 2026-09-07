# プラグイン API リファレンス

署名だけを眺めるための頁。使い方の流れは概念頁を見ること:
[実行モデルと寿命](ja/plugins/lifecycle.md)・
[manifest と能力・承認](ja/plugins/manifest.md)・
[ビルドと配布・してはいけないこと](ja/plugins/build.md)。
初めて作るときは [環境構築と書き方](ja/plugins/setup.md) から入ること。

- イベント購読 API は無い。詳しくは [イベント](#イベント) を見ること。

## 目次

- [`ctx`](#ctx): パッチが受け取る文脈
- [`ui`](#ui): `patch`・`exists`・`wrap`・`after`・`before`・`settings`・`stack`・`node`・`text`・`badge`・`button`・`icon`
- [`log`](#log): `info`・`warn`・`error`
- [`storage`](#storage): `get`・`set`・`remove`・`getJSON`・`setJSON`
- [インターフェース](#インターフェース): `UINode`・`NewUINode`・`PatchContext`・`PatchFn`・データ8種
- [型エイリアスと列挙的型](#型エイリアスと列挙的型): `NodeId`・`PluginNodeId`・`CreatableNodeId`・`CoreCreatableNodeId`・`NodeState`・`DataByNode`
- [イベント](#イベント): 購読 API なし

## `ctx`

パッチが受け取る文脈。`{ data }` の形で渡され、パッチ関数ごとに
そのノードぶんだけが取り出される。

- `ctx.data` はノード毎のドメイン事実で、ID から型が付く
  (`ctx.data.author.bot` 等)。該当なしは `undefined`。
- 読取専用で、生ペイロードは出ない。追加は非破壊、削除改名は破壊的
  (ABI の一部)。
- どの ID がどの型を持つかは [安定 ID カタログ](ja/theme/ids.md) の
  `data` 列を見ること。型の一覧は [インターフェース](#インターフェース)
  にある。

```ts
ui.patch("chat.message.header.author", (node, ctx) => {
  if (!ctx.data.author.bot) return node;  // data を見て分岐
  return ui.after(node, ui.badge({ text: "BOT" }));
});
```

<!-- BEGIN GENERATED: api -->

## `ui`

### `ui.patch`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
```

安定 ID に変形を登録する。走査はボトムアップで、照合は適用前の ID に対して行い、出力には再帰しない。存在しない ID への登録は無害。`fn` は純粋に書く。

```ts
ui.patch("chat.message.header.author", (node) =>
  ui.after(node, ui.badge({ text: "hi" })),
);
```

### `ui.exists`

```ts
exists(id: NodeId): boolean
```

その ID がこの環境に存在しうるか。登録自体は存在しない ID でも無害なので、事前分岐のためだけに使う。

### `ui.wrap`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

ノードを子に包んで返す。包みに core ID (`app.*` / `chrome.*` / `nav.*` / `chat.*`) は使えない。

### `ui.after`

```ts
after(node: UINode, sibling: UINode): UINode
```

後に兄弟を足す。`layout.row` で包んで返す。出力は最終形として扱われる。

### `ui.before`

```ts
before(node: UINode, sibling: UINode): UINode
```

前に兄弟を足す。`layout.row` で包んで返す。出力は最終形として扱われる。

### `ui.settings`

```ts
settings(fn: () => UINode): void
```

表示専用の設定頁を登録する。操作部品は置いても押せない。保存は設定画面の側で行う。

### `ui.stack`

```ts
stack(nodes: UINode[]): UINode
```

縦に積む。`layout.column` を返す。

### `ui.node`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

自名前空間のノードを作る。core ID (`app.*` 等) を作るとその出力ごと捨てられる。

### `ui.text`

```ts
text(value: string): NewUINode
```

`primitive.text` を作る。表示文字列を持つノードを返す。

### `ui.badge`

```ts
badge(opts: { text: string; tone?: string }): NewUINode
```

`primitive.badge` を作る。`tone` 等の見た目指定はテーマの持ち物のため落とす。

### `ui.button`

```ts
button(opts: { label: string; onPress: () => void }): NewUINode
```

`primitive.button` を作る。`onPress` 等の関数は境界を越えられず黙って落とす。設定画面では押せない。

### `ui.icon`

```ts
icon(name: string): NewUINode
```

`primitive.icon` を作る。絵文字名を持つノードを返す。

## `log`

### `log.info`

```ts
info(msg: string): void
```

記録に残す (情報)。引数は記録文。

### `log.warn`

```ts
warn(msg: string): void
```

記録に残す (警告)。引数は記録文。

### `log.error`

```ts
error(msg: string): void
```

記録に残す (異常)。引数は記録文。

## `storage`

### `storage.get`

```ts
get(key: string): string | null
```

鍵を読む。不在で `null` を返す。

### `storage.set`

```ts
set(key: string, value: string): void
```

鍵に書く。即時保存される。パッチの中から呼ばない。

### `storage.remove`

```ts
remove(key: string): void
```

鍵を消す。即時保存される。パッチの中から呼ばない。

### `storage.getJSON`

```ts
getJSON<T>(key: string, fallback: T): T
```

JSON で読む。不在時と壊れ JSON の両方で `fallback` を返す。パッチの中から呼ばない。

### `storage.setJSON`

```ts
setJSON(key: string, value: unknown): void
```

JSON 化して書く。パッチの中から呼ばない。

## インターフェース

### `UINode`

```ts
interface UINode {
  /** The stable ID. */
  id: NodeId | PluginNodeId;
  /** Distinguishes siblings sharing an id under one parent. Read-only. */
  readonly key?: string;
  /** The states currently held. */
  readonly states?: readonly NodeState[];
  /**
   * The colour the data carries (`#RRGGBB`): a role colour, a folder colour.
   *
   * Not a style. Where it lands is the theme's choice, and it only fills a
   * property written as `$data.tint`.
   */
  readonly tint?: string;
  props?: Record<string, unknown>;
  children?: UINode[];
}
```

画面上の部品 1 個。`key`・`states`・色・参照は読取専用で、出力側で変えられない。

### `NewUINode`

```ts
interface NewUINode extends UINode {
  id: CreatableNodeId;
}
```

作る側のノード。`id` が作れる ID に限られる以外は `UINode` と同じ。

### `PatchContext`

```ts
interface PatchContext<Id extends NodeId = NodeId> {
  readonly data: Id extends keyof DataByNode ? Readonly<DataByNode[Id]> : undefined;
}
```

パッチが受け取る文脈。`data` は ID から型が付き、該当なしは `undefined`。読取専用。

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

1 人ぶんの像。名前・表示名・BOT 判別など。

欄: `id`・`username`・`displayName`・`bot`・`avatarUrl?`

### `MessageData`

```ts
interface MessageData {
  readonly id: string;
  readonly channelId: string;
  readonly guildId?: string;
  readonly createdAt: string;
  readonly editedAt?: string;
  /** Plain text. Parsed Markdown appears as nodes. */
  readonly content: string;
  readonly author: UserData;
  readonly pinned: boolean;
  readonly referencedMessageId?: string;
}
```

メッセージ 1 件。`content` は素文で、装飾済み本文はノード側にある。

欄: `id`・`channelId`・`guildId?`・`createdAt`・`editedAt?`・`content`・`author`・`pinned`・`referencedMessageId?`

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

ギルド 1 個と未読状態。

欄: `id`・`name`・`iconUrl?`・`unread`・`mentionCount`

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

チャンネル 1 個と未読状態。

欄: `id`・`name`・`type`・`topic?`・`nsfw`・`unread`・`mentionCount`

### `CategoryData`

```ts
interface CategoryData {
  readonly id: string;
  readonly name: string;
  readonly collapsed: boolean;
}
```

カテゴリ 1 個。

欄: `id`・`name`・`collapsed`

### `DmData`

```ts
interface DmData {
  readonly id: string;
  readonly recipients: readonly UserData[];
  readonly unread: boolean;
  readonly mentionCount: number;
}
```

DM 1 件。

欄: `id`・`recipients`・`unread`・`mentionCount`

### `MemberData`

```ts
interface MemberData {
  readonly user: UserData;
  /** Their name in this guild, or `user.displayName` if unset. */
  readonly displayName: string;
  /** `online` / `idle` / `dnd` / `offline` */
  readonly status: string;
  readonly roles: readonly string[];
}
```

名簿の 1 人。`roles` は ID ではなく名列。

欄: `user`・`displayName`・`status`・`roles`

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

添付 1 件。

欄: `id`・`filename`・`size`・`contentType?`・`url`・`width?`・`height?`

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

埋め込み 1 件。

欄: `type`・`title?`・`description?`・`url?`・`color?`

## 型エイリアスと列挙的型

### `NodeId`

```ts
type NodeId =
  | "app.root"
  | "app.window"
  | /* ... 118 more */
  | "layout.scrollbar.thumb"
  ;
```

安定 ID の合併型 (121 個)。未知 ID は型で通らない。一覧は安定 ID カタログを見ること。

一覧は[安定 ID カタログ](ja/theme/ids.md)を見ること。

### `PluginNodeId`

```ts
type PluginNodeId = `plugin.${string}`;
```

自名前空間の ID。`plugin.`＋自 ID の `.`→`_`。互換性の維持は作者の責任である。

### `CreatableNodeId`

```ts
type CreatableNodeId = CoreCreatableNodeId | PluginNodeId;
```

プラグインが作れる ID。`app.*` / `chrome.*` / `nav.*` / `chat.*` は作れない。

### `CoreCreatableNodeId`

```ts
type CoreCreatableNodeId =
  | "overlay.layer"
  | "overlay.scrim"
  | /* ... 41 more */
  | "layout.scrollbar.thumb"
  ;
```

作れる core 側 ID の合併型。`overlay.*`・`settings.*`・`primitive.*`・`layout.*` (44 個)。

### `NodeState`

```ts
type NodeState =
  | "hover"
  | "active"
  | "focus"
  | "selected"
  | "disabled"
  | "unread"
  | "mentioned"
  | "loading"
  | "grouped"
  | "collapsed";
```

テーマが条件にできる状態と同じ集合 (10 個)。

### `DataByNode`

```ts
interface DataByNode {
  "nav.guild_list.item": GuildData;
  "nav.guild_list.item.icon": GuildData;
  "nav.guild_list.item.pill": GuildData;
  "nav.guild_list.item.badge": GuildData;
  "nav.channel_list.category": CategoryData;
  "nav.channel_list.item": ChannelData;
  "nav.channel_list.item.icon": ChannelData;
  "nav.channel_list.item.name": ChannelData;
  "nav.channel_list.item.badge": ChannelData;
  "nav.dm_list.item": DmData;
  "nav.member_list.item": MemberData;
  "nav.member_list.item.avatar": MemberData;
  "nav.member_list.item.presence": MemberData;
  "nav.member_list.item.name": MemberData;
  "chat.header": ChannelData;
  "chat.header.title": ChannelData;
  "chat.header.topic": ChannelData;
  "chat.message": MessageData;
  "chat.message.avatar": MessageData;
  "chat.message.header": MessageData;
  "chat.message.header.author": MessageData;
  "chat.message.header.badges": MessageData;
  "chat.message.header.timestamp": MessageData;
  "chat.message.reply_ref": MessageData;
  "chat.message.content": MessageData;
  "chat.message.attachments": MessageData;
  "chat.message.attachment": AttachmentData;
  "chat.message.embeds": MessageData;
  "chat.message.embed": EmbedData;
  "chat.message.actions": MessageData;
}
```

ID からデータ型への対応表。`ctx.data` の型付けの根拠である。対応が無い ID は `undefined`。

<!-- END GENERATED: api -->

## イベント

出来事を購読する API は無い。出来事への反応はパッチで行う:

- 画面外のノードは訪れないため、全件走査はできない。
- `ui.exists` で環境分岐し、該当ノードのパッチで変形する。
- パッチは純粋に保ち、通信・保存・計数は持ち込まない。

Gateway 出来事のミドルウェアは将来の仕組みであり、来たらこの節に足す。
