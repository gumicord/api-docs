# テーマ API

プログラミングの知識は要らない。JSON が書ければ作れる。CSS の知識も
要らない (使わないため)。

## はじめに

テーマはクライアントの見た目を決める JSON (`theme.json`) である。
考え方は3つだけ:

- **トークン**: 色や寸法の名前付き置き場 (絵の具皿)。
- **ルール**: 「どの部品を」「どう塗るか」の組。部品は安定 ID で指名する。
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

## 目次

- [環境構築と書き方](ja/theme/setup.md) — 道具の用意・初回の手順・サンプル
- [安定 ID カタログ](ja/theme/ids.md) — `select` に書ける ID と役目の一覧
- [manifest とトークン](ja/theme/manifest-tokens.md) — 身元・値の表・`$data.tint`
- [ルールと条件](ja/theme/rules.md) — `select`・`when`・カスケード・選択
- [スタイル](ja/theme/styles.md) — プロパティ・`transition`・行内装飾スロット
- [背景とアセット・壊れたとき](ja/theme/assets.md) — 背景画像・参照形式・検証
