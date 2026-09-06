# manifest とトークン

## manifest

| フィールド | 必須 | 意味・形 |
|---|---|---|
| `id` | 必須 | 逆ドメイン (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`) |
| `name` | 必須 | 1〜64 文字。表示名 |
| `version` | 必須 | セマンティック版 (`X.Y.Z`、予告可) |
| `abi` | 必須 | 1 以上の整数。想定する UITree ABI メジャー版 |
| `author` | 任意 | 128 文字まで |
| `description` | 任意 | 512 文字まで |
| `homepage` | 任意 | URI |
| `license` | 任意 | 64 文字まで |
| `remoteAssets` | 任意 | 外部アセットのホスト名配列 (16 件まで)。未宣言ホストへは取得しない |

`abi` がクライアントより大きいと「新しすぎる」と伝え、既知のルールのみ
適用する。小さい場合は通常どおり適用する。

## トークン

使い回す値の表。名前は自由 (`カテゴリ.用途.変種` の順を推奨)。
クライアントが名前を要求することはない。

| 値の型 | 書式 | 例 |
|---|---|---|
| 色 | `#RGB` / `#RRGGBB` / `#RRGGBBAA` | `"#7c6cf0"`、`"#ffffff14"` |
| 長さ | 数値。論理 px (DPI 換算はレンダラ) | `8` |
| フォント | オブジェクト | `{ "family": "Inter", "size": 15, "lineHeight": 22, "weight": 600 }` |
| 影 | オブジェクト | `{ "x": 0, "y": 2, "blur": 8, "color": "#00000040" }` |
| 時間 | 数値 (ミリ秒) | `150` |

フォントの欄: `family` (省略時は同梱既定。省略推奨)・`size`・
`lineHeight`・`weight` (100〜900)・`italic`・`letterSpacing`。
影の欄: `x`・`y`・`blur`・`spread`・`color` (必須)。

`$名前` で参照する。トークン同士の参照可。循環はエラー。

```jsonc
{ "background": "$color.bg.raised" }
{
  "color.brand": "#7c6cf0",
  "color.accent": "$color.brand"
}
```

長さは常に論理 px で書く。物理 px への換算はレンダラが行う。

## `$data.tint`

トークンではなく、ノードが持ってきた色 (役職色・フォルダ色) を指す印。
書けるのは `color`・`borderColor`・`background` のみ。色を持たない
ノードでは何も起きず、後の素の色ルールが勝つ。識別子は渡らない
(特定サーバだけを狙えない)。

```jsonc
// 既定の色。色を持たない人はここで止まる
{ "select": "nav.member_list.item.name", "style": { "color": "$color.text.secondary" } },
// 色を持っている人だけ、その色になる
{ "select": "nav.member_list.item.name", "style": { "color": "$data.tint" } }
```

| | |
|---|---|
| 書ける場所 | `color` / `borderColor` / `background` |
| 色を持たないノード | 何も起きない。前のルールが書いた値が残る |
| 後から素の色を書いたルール | そちらが勝つ。印は下りる |

色を載せるノードは `tint` を持つものに限られる。いまは
`nav.guild_list.folder` / `nav.guild_list.folder.icon` (フォルダの色) と
`nav.member_list.item.name` (役職の色) である。
