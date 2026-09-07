# 環境構築と書き方

## 道具

専用の道具は要らない。テキストエディタがあれば書ける。Node.js も
ビルドも要らない (`theme.json` はそのまま読まれる)。

あるとよいもの:

- JSON Schema 対応のエディタ。先頭に
  `"$schema": "https://gumicord.dev/schema/theme-1.json"` と書くと
  入力補完と型検査が効く (無くても動く)。

## 配置

`themes/` の下に 1 フォルダ作り、`theme.json` を置く。同梱の絵や書体は
同じフォルダの `assets/` に入れる。

```
themes/
└─ mytheme/
   ├─ theme.json
   └─ assets/
      └─ wallpaper.png
```

## 開発ループ

1. `GUMICORD_THEME=themes/mytheme/theme.json` で差し替えて起動する
   (一覧には出ない開発用の上書き)。
2. 保存すれば再起動なしに再適用される。
3. 編集中の破損では直前の成功版が維持される。壊れた箇所は設定画面の
   一覧に JSON パス付きで出る。
4. 気に入ったら設定画面のテーマ一覧から選ぶ。選択は
   `themes/active.json` (`{"theme": "<manifest id>"}`) に残る。

## 書き方

小さく始める。書かない部分は変えないのが規則である。

1. `manifest` に身元を書く (`id` は世界で1つ。逆ドメイン)。
2. 使う色と寸法を `tokens` に名前を付けて置く (絵の具皿)。
3. 塗りたい部品を安定 ID で指名し (`select`)、`style` を書く。
   ID は [安定 ID カタログ](ja/theme/ids.md) から引く。
4. 条件が要れば `when` を足す (`state`・`platform` 等)。
   詳しくは [ルールと条件](ja/theme/rules.md) を見ること。
5. 保存して画面で確かめる。

## サンプル

全文例 (1 行ずつ読む):

```jsonc
{
  "$schema": "https://gumicord.dev/schema/theme-1.json",  // 入力補完用。無くても動く
  "manifest": { "id": "dev.example.mytheme", "name": "My theme",
                "version": "1.0.0", "abi": 1 },  // 身元
  "tokens": {
    "color.bg.base": "#101018",      // 窓の地色
    "color.bg.hover": "#ffffff14",   // 行ホバーの明かり
    "color.text.primary": "#f0f0f5", // 本文色
    "color.accent": "#7c6cf0"        // 差し色
  },
  "rules": [
    // 窓全体の背景を地色に。$名前 でトークンを引く
    { "select": "app.window", "style": { "background": "$color.bg.base" } },
    // メッセージ本文を本文色に
    { "select": "chat.message.content",
      "style": { "color": "$color.text.primary" } },
    // 作者名を差し色に
    { "select": "chat.message.header.author",
      "style": { "color": "$color.accent" } },
    // マウスが乗った行だけ薄く光らせる (条件付きの例)
    { "select": "chat.message",
      "when": { "state": "hover" },
      "style": { "background": "$color.bg.hover" } },
    // 狭い画面ではサーバ列を丸く小さくする (携帯向けの例)
    { "select": "nav.guild_list.item",
      "when": { "platform": "mobile" },
      "style": { "width": 40, "height": 40 } }
  ]
}
```

この例で変わるもの: 窓の地色、本文と作者名の文字色、行ホバーの明かり、
携帯でのサーバ列の大きさ。他は標準のまま残る。

次に読むもの: [manifest とトークン](ja/theme/manifest-tokens.md)・
[スタイル](ja/theme/styles.md)・[背景とアセット](ja/theme/assets.md)。
