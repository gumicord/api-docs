# テーマ API

プログラミングの知識は要らない。JSON が書ければ作れる。CSS の知識も
要らない (使わないため)。

## はじめに

テーマはクライアントの見た目を決める JSON (`theme.json`) である。
考え方は3つだけ:

- **トークン**: 色や寸法の名前付き置き場 (絵の具皿)。`color.bg.base`
  のような名前を付けて使い回す。
- **ルール**: 「どの部品を」「どう塗るか」の組。部品は名前
  (`chat.message.content` のような安定 ID) で指名する。
- **そのまま出る**: 保存すれば即座に画面に反映される。壊れても画面が
  白紙になることはない (壊れた箇所だけ無視される)。

動くまでの手順:

1. `themes/` の下に 1 フォルダ作り (`mytheme/`)、`theme.json` を置く。
   同梱の絵や書体があれば同じフォルダの `assets/` に入れる。
2. 下の最小例を写す。
3. 開発中は `GUMICORD_THEME=themes/mytheme/theme.json` で差し替えて確かめる。
4. 気に入ったら設定画面のテーマ一覧から選ぶ (開発中の上書きは一覧に出ない)。

最小例 (1 行ずつ読む):

```jsonc
{
  "$schema": "https://gumicord.dev/schema/theme-1.json",  // 入力補完用。無くても動く
  "manifest": { "id": "dev.example.mytheme", "name": "My theme",
                "version": "1.0.0", "abi": 1 },  // 身元。id は世界で1つ
  "tokens": {
    "color.bg.base": "#101018",      // 窓の地色に使う色に名前を付ける
    "color.text.primary": "#f0f0f5"  // 本文色に使う色に名前を付ける
  },
  "rules": [
    // 窓全体の背景を地色に。$名前 でトークンを引く
    { "select": "app.window", "style": { "background": "$color.bg.base" } },
    // メッセージ本文を本文色に
    { "select": "chat.message.content",
      "style": { "color": "$color.text.primary" } },
    // マウスが乗った行だけ薄く光らせる (条件付きの例)
    { "select": "chat.message",
      "when": { "state": "hover" },
      "style": { "background": "#ffffff14" } }
  ]
}
```

この例で変わるもの: 窓の地色、本文の文字色、行ホバーの明かり。
他は標準のまま残る (書かない部分は変えないのが規則)。

トップレベルに置けるのは `$schema`・`manifest`・`tokens`・`rules` のみ。

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

### `$data.tint`

トークンではなく、ノードが持ってきた色 (役職色・フォルダ色) を指す印。
書けるのは `color`・`borderColor`・`background` のみ。色を持たない
ノードでは何も起きず、後の素の色ルールが勝つ。識別子は渡らない
(特定サーバだけを狙えない)。

## ルール

`select` と `style` が必須、`when` は任意。余計なキーは不可。
適用は記述順 (後の勝ち、プロパティ単位)。詳細度は無い。

- `select` は安定 ID の完全一致のみ。ワイルドカードは無い。ID 一覧は
  クライアントの安定 ID 表で確認する (`chat.message.content` 等)。

### `when`

全キーの AND。ただし `state` 配列は AND (全件成立が必要)、
プラットフォーム配列は OR (いずれかでよい)。

| キー | 形 |
|---|---|
| `state` | 状態名またはその配列。`hover`・`active`・`focus`・`selected`・`disabled`・`unread`・`mentioned`・`loading`・`grouped`・`collapsed` |
| `platform` | `windows`・`macos`・`linux`・`android`・`ios`・`desktop`・`mobile` のいずれかまたは配列 |
| `colorScheme` | `light`・`dark` (OS 設定に従う) |
| `minWidth`・`maxWidth` | 数値。窓幅の両端を含む条件 (論理 px) |
| `slot` | 文字列。同 ID 兄弟の位置区別のみ。スノーフレークには一致しない (特定鯖・特定者を塗れない。テーマを配れるものに保つため) |

例: `{ "select": "nav.guild_list.item", "when": { "platform": "mobile" },
"style": { "width": 40, "height": 40 } }`。

未知のキー・状態名・プラットフォーム名があるルールは全体を無視する
(条件だけ落とすと適用範囲が広がるため)。

## スタイル

書式・既定・継承・補間をプロパティ毎に示す。未知プロパティは警告して
無視する (前方互換のため)。

| プロパティ | 値の形 | 既定・継承・補足 |
|---|---|---|
| `background` | 色・背景オブジェクト・`$` 参照 | 既定は透明。下から色→画像→tint の合成 |
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

### `transition`

動くのは色 (`color`・`borderColor`・`background` の色)・`radius`・
`borderWidth`・`opacity`・寸法・`gap` のみ。書体・影・画像は即切替。
ease-out 固定、子へ継承なし、書いたノードだけが動く。片方が未指定の
値は混ぜない。時間駆動のため 60Hz/144Hz で一致する。

### 行内装飾スロット

`primitive.text` の `when.slot` で付ける。`bold` (`**`)・`italic`
(`*`)・`underline` (`__`)・`strike` (`~~`)・`spoiler` (`||`。隠れ中は
`color` が塗りになる)・`code` (`` ` ``)・`link`・`mention`・`h1`・
`h2`・`h3`・`subtext`・`bullet`・`quote_bar`・`li0`〜`li4` (箇条書きの
深さ。幅はテーマが決める)。重なりは書いた順に重ねる。

## 背景とアセット

`image` の他に `fit` (`cover` 既定・`contain`・`stretch`・`tile`・
`none`)、`position` (0.0〜1.0 の2要素、既定中央)、`opacity` (既定
1.0)、`blur` (読込時適用、上限 256) を持つ。任意の安定 ID に書ける。
窓いっぱいの絵を見せたいときは `chat.message_list` と `chat.view` に
背景色を書かない (透明にして下の窓絵を通す)。

参照は 3 形式:

- テーマ相対パス。同梱は `assets/` 下推奨。`../` 脱出不可。
  `png|jpg|jpeg|webp|avif|woff2|ttf|otf` のみ。
- `data:` 埋め込み。画像 4 形式と `font/woff2` のみ。
- `https:` 外部。`remoteAssets` 宣言＋承認が要る。拒否や失敗時は
  `background.color` に落ちる (テーマ全体は止めない)。

Cookie 等は送らず、リダイレクトで宣言外へは追わず、取得はキャッシュし、
1 ファイル 32MB まで。

## 壊れたとき

全体を捨てるのは JSON 構文破損とマニフェスト不正のみ。それ以外は箇所
単位で捨てる:

| 状態 | 落とす範囲 | 知らせ |
|---|---|---|
| トークン循環・未定義参照・型違い | 当該プロパティ (循環は参照元ルール) | エラー |
| 未知の安定 ID・プロパティ・条件 | ルールまたは当該箇所 | 警告 (前方互換のため) |
| 取れないアセット | `background.color` に代替 | 警告 (未承認は「未承認」表示) |

設定画面の一覧に位置 (JSON パス) 付きで出す。起動を妨げない。

## 選択と更新

- 選択は `themes/active.json` (`{"theme": "<manifest id>"}`)。無ければ標準テーマ。
- 標準は同梱 Midnight (ファイルを持たないためホットリロード対象外)。
- `GUMICORD_THEME` の指すファイルが最優先 (一覧には出ない)。
- 読めないテーマは一覧に出さず、適用せず、直前を維持して知らせる。
- ファイル変更は再起動なしに再適用する。編集中の破損では直前の成功版を維持する。
