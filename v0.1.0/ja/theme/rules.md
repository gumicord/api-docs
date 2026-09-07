# ルールと条件

`select` と `style` が必須、`when` は任意。余計なキーは不可。
適用は記述順 (後の勝ち、プロパティ単位)。詳細度は無い。

ID 一覧は [安定 ID カタログ](ja/theme/ids.md) で確認する。

## `select`

安定 ID の完全一致のみ。ワイルドカード・子孫指定は無い。

```jsonc
{ "select": "chat.message.header.author", "style": { /* ... */ } }
```

追加された ID に既存テーマの見た目が変わらないよう、
パターン一致は提供しない。前方互換性を採る。

## `when`

全キーの AND。ただし `state` 配列は AND (全件成立が必要)、
プラットフォーム配列は OR (いずれかでよい)。

| キー | 形 |
|---|---|
| `state` | 状態名またはその配列。`hover`・`active`・`focus`・`selected`・`disabled`・`unread`・`mentioned`・`loading`・`grouped`・`collapsed` |
| `platform` | `windows`・`macos`・`linux`・`android`・`ios`・`desktop`・`mobile` のいずれかまたは配列 |
| `colorScheme` | `light`・`dark` (OS 設定に従う) |
| `minWidth`・`maxWidth` | 数値。窓幅の両端を含む条件 (論理 px) |
| `slot` | 文字列。同 ID 兄弟の位置区別のみ。スノーフレークには一致しない (特定鯖・特定者を塗れない。テーマを配れるものに保つため) |

例:

```jsonc
{ "select": "nav.guild_list.item", "when": { "platform": "mobile" },
  "style": { "width": 40, "height": 40 } }
{
  "select": "nav.channel_list.item",
  "when": { "state": ["hover", "unread"], "colorScheme": "dark" },
  "style": { /* ... */ }
}
```

状態は同時に複数立ちうるため、配列は「両方」の意味になる。
プラットフォームは同時に 1 つしか成立しないため、配列は「どちらか」の
意味になる。どちらもその書き方に意味がある唯一の解釈を採っている。

`slot` から見えるのは位置のみである。`Id` (スノーフレーク) に一致すると
「このサーバだけ赤くする」と書けてしまい、テーマが配れるもので
なくなる。`Index` は並びが変われば別のものを指すため飾り分けに使わない。

```json
{ "select": "nav.user_panel.presence", "when": { "slot": "dnd" },
  "style": { "background": "#e05260" } }
```

未知のキー・状態名・プラットフォーム名があるルールは全体を無視する
(条件だけ落とすと適用範囲が広がるため)。

## カスケード

規則は 1 つだけ: **記述順に適用し、後のルールが前を上書きする。**

```jsonc
"rules": [
  { "select": "chat.message", "style": { "background": "#111" } },
  { "select": "chat.message", "when": { "state": "hover" }, "style": { "background": "#222" } }
]
// hover 中は #222。順序を逆にすると hover が効かなくなる。
```

上書きの単位はプロパティである:

```jsonc
{ "select": "chat.message", "style": { "background": "#111", "radius": 8 } },
{ "select": "chat.message", "style": { "background": "#222" } }
// 結果: background=#222, radius=8
```

書いた順がすべてであるため、「なぜ効かないか」は上から読めば分かる。

プラグインがテーマより後に適用される。衝突したらプラグインが勝つ。

## 選択

- 選択は `themes/active.json` (`{"theme": "<manifest id>"}`)。無ければ標準テーマ。
- 標準は同梱 Midnight (ファイルを持たないためホットリロード対象外)。
- `GUMICORD_THEME` の指すファイルが最優先 (一覧には出ない)。
- 読めないテーマは一覧に出さず、適用せず、直前を維持して知らせる。
- ファイル変更は再起動なしに再適用する。編集中の破損では直前の成功版を維持する。
