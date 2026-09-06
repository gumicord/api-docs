# `ctx.data` とノードの形

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

どの ID がどの型を持つかは [安定 ID カタログ](ja/theme/ids.md) の
`data` 列を見ること。

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

| 対象 | `props` | 描く内容 |
|---|---|---|
| `primitive.text`・`primitive.badge`・`primitive.button`・`plugin.*` | `value` / `text` / `label` の順にある最初の文字列 | その文字列 |
| `primitive.icon` | `name` 文字列 | その名の絵 |
| `primitive.image` | `url` 文字列 | その URL の絵 |
| それ以外 | — | 描く内容なし (子は描く) |

`onPress` のような関数は境界を越えられず、黙って落とす。`tone` や `gap`
のような見た目の指定はテーマの持ち物なので落とす。どちらも警告は出さない
— 正しいプラグインが毎フレーム警告を出すようでは記録が読めなくなる。
