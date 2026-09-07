# スタイル

書式・既定・継承・補間をプロパティ毎に示す。未知プロパティは警告して
無視する (前方互換のため)。

| プロパティ | 値の形 | 既定・継承・補足 |
|---|---|---|
| `background` | 色・背景オブジェクト・`$` 参照 | 既定は透明。下から色→画像→tint の合成。詳しくは [背景とアセット](ja/theme/assets.md) |
| `color` | 色・`$` 参照・`$data.tint` | 子へ継承する。文字色 |
| `font` | フォント・`$` 参照 | 子へ継承する |
| `borderColor` | 色・`$` 参照・`$data.tint` | 罫線色 |
| `borderWidth` | 長さ・`$` 参照 | 罫の太さ |
| `radius` | 長さ・`$` 参照 | 角丸。補間可 |
| `padding`・`margin` | 長さまたは `[上, 右, 下, 左]` | 内側・外側の間隔 |
| `gap` | 長さ・`$` 参照 | 子の間隔。補間可 |
| `width`・`height` | 長さ・`$` 参照 | 補間可 (周囲配置も動く) |
| `minWidth`・`maxWidth`・`minHeight`・`maxHeight` | 長さ・`$` 参照 | 制約 (`when` の同名とは別物) |
| `opacity` | 0.0〜1.0 | 補間可 |
| `shadow` | 影・`$` 参照 | 即切替 |
| `transition` | ミリ秒 | 追従時間。自体は何も描かない |
| `decoration` | `none`・`underline`・`strikethrough` の空白区切り | 未知語が1つでもあれば全体を捨てる |

継承するのは `color` と `font` のみ。配置の上書きは M2 で、M1 の
テーマは見た目だけを変える。

## `transition`

動くのは色 (`color`・`borderColor`・`background` の色)・`radius`・
`borderWidth`・`opacity`・寸法・`gap` のみ。書体・影・画像は即切替。
ease-out 固定、子へ継承なし、書いたノードだけが動く。片方が未指定の
値は混ぜない。時間駆動のため 60Hz/144Hz で一致する。

```jsonc
{ "select": "nav.guild_list.item.icon",
  "style": { "radius": 8, "transition": 150 } },
{ "select": "nav.guild_list.item.icon",
  "when": { "state": "hover" }, "style": { "radius": 4 } }
```

書いたノードだけが動く。既定で全部を動かすと一覧をめくるたびに何十行も
動き出す。寸法が動くと周りの配置もそのぶん動く。

## 行内装飾スロット

`primitive.text` の `when.slot` で付ける。重なりは書いた順に重ねる。

| slot | いつ |
|---|---|
| `bold`・`italic`・`underline`・`strike` | `**`・`*`・`__`・`~~` |
| `spoiler` | `||`。隠れ中は `color` が塗りになる |
| `code` | `` ` `` (行の中) |
| `link`・`mention` | リンク・裸の URL / `<@1>`・`<#1>`・`<@&1>`・`@everyone` |
| `h1`・`h2`・`h3`・`subtext` | 見出しと `-# ` |
| `bullet` | 箇条書きの印 |

```json
{ "select": "primitive.text", "when": { "slot": "underline" },
  "style": { "decoration": "underline" } },
{ "select": "primitive.text", "when": { "slot": "strike" },
  "style": { "decoration": "strikethrough", "color": "$color.text.muted" } }
```

縦に積まれるものは通常どおり安定 ID で狙う: 引用の線は
`primitive.divider` の `slot: "quote_bar"`、コードブロックは
`primitive.code_block`、箇条書きの字下げは `layout.row` の
`slot: "li0"`〜`"li4"` (幅はテーマが決める)。
