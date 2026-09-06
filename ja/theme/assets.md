# 背景とアセット・壊れたとき

## 背景オブジェクト

`image` の他に `fit`・`position`・`opacity`・`blur` を持つ。任意の安定 ID
に書ける。

| キー | 型 | 既定 | 意味 |
|---|---|---|---|
| `color` | 色 | 透明 | 画像の下に敷く色。読めなかった場合の代替でもある |
| `image` | アセット参照 | — | 背景画像 |
| `fit` | `cover` / `contain` / `stretch` / `tile` / `none` | `cover` | 領域への合わせ方 |
| `position` | `[x, y]` (各 0.0〜1.0) | `[0.5, 0.5]` | 寄せ位置 |
| `opacity` | 0.0〜1.0 | `1.0` | 画像の不透明度 |
| `blur` | 数値 (論理 px) | `0` | ぼかし半径。読込時に一度だけ適用し、結果をキャッシュする |
| `tint` | 色 | 透明 | 画像の上に重ねる色。可読性確保に使う |

合成順序は下から `color` → `image` → `tint`。

窓いっぱいの絵を見せたいときは `chat.message_list` と `chat.view` に
背景色を書かない (透明にして下の窓絵を通す)。

```jsonc
{
  "select": "app.window",
  "style": {
    "background": {
      "color": "#0f0f17",
      "image": "assets/wallpaper.png",
      "fit": "cover",
      "tint": "#0f0f1740"
    }
  }
}
```

半透明合成では上に載る UI 側を 8 桁色 (`#RRGGBBAA`) で透かす:

```jsonc
{ "select": "app.window",      "style": { "background": { "image": "assets/bg.png", "fit": "cover" } } },
{ "select": "nav.channel_list","style": { "background": "#16161fcc" } },
{ "select": "chat.header",     "style": { "background": "#0f0f1799" } }
```

## 参照は 3 形式

| 種類 | 書式 | 例 |
|---|---|---|
| 同梱アセット | `theme.json` からの相対パス | `"assets/wallpaper.png"` |
| データ URI | `data:` | `"data:image/png;base64,..."` |
| 外部 URL | `https:` のみ | `"https://cdn.example.com/bg.png"` |

- テーマ相対パス。同梱は `assets/` 下推奨。`../` 脱出・絶対パス・
  シンボリックリンク追跡は不可。`png|jpg|jpeg|webp|avif|woff2|ttf|otf` のみ。
- `data:` 埋め込み。画像 4 形式と `font/woff2` のみ。
- `https:` 外部。`remoteAssets` 宣言＋承認が要る。拒否や失敗時は
  `background.color` に落ちる (テーマ全体は止めない)。

```
midnight/
├─ theme.json
└─ assets/
   ├─ wallpaper.png
   └─ Inter.woff2
```

外部取得の制約: 認証情報を付けない (Cookie 等を送らない)・`https` のみ・
取得はキャッシュし、起動のたびに再取得しない・1 ファイル 32MB まで・
宣言外へのリダイレクトは追わない。同梱アセットを推奨する。

## 壊れたとき

全体を捨てるのは JSON 構文破損とマニフェスト不正のみ。それ以外は箇所
単位で捨てる:

| 状態 | 落とす範囲 | 知らせ |
|---|---|---|
| トークン循環・未定義参照・型違い | 当該プロパティ (循環は参照元ルール) | エラー |
| 未知の安定 ID・プロパティ・条件 | ルールまたは当該箇所 | 警告 (前方互換のため) |
| 取れないアセット | `background.color` に代替 | 警告 (未承認は「未承認」表示) |

設定画面の一覧に位置 (JSON パス、例 `rules[3].style.background.image`)
付きで出す。起動を妨げない。
