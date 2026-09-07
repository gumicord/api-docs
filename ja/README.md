# Gumicord API ドキュメント

テーマとプラグインの作者向けリファレンス。画面の部品はすべて
**安定 ID** (`chat.message.content` のような名前) を持ち、テーマは
ID を指名して見た目を決め、プラグインは ID を指名して変形を登録する。
ID と役目の一覧は [安定 ID カタログ](ja/theme/ids.md) にある。

## どちらを読むか

- 見た目を変えたい → [テーマ API](ja/theme.md)。JSON が書ければ作れる。
  初めて作るときは [環境構築と書き方](ja/theme/setup.md) から入ること。
- 画面に部品を足したい → [プラグイン API](ja/plugins.md)。
  TypeScript が読める人向け。初めて作るときは
  [環境構築と書き方](ja/plugins/setup.md) から入ること。
- 署名だけ眺めたい → [プラグイン API リファレンス](ja/plugins/reference.md)。

## 目次

- [テーマ API](ja/theme.md) — 概要・最小例・子ページへの案内
  - [環境構築と書き方](ja/theme/setup.md) — 道具の用意・初回の手順・サンプル
  - [安定 ID カタログ](ja/theme/ids.md) — `select` に書ける ID と役目の一覧
  - [manifest とトークン](ja/theme/manifest-tokens.md) — 身元・値の表・`$data.tint`
  - [ルールと条件](ja/theme/rules.md) — `select`・`when`・カスケード・選択
  - [スタイル](ja/theme/styles.md) — プロパティ・`transition`・行内装飾スロット
  - [背景とアセット・壊れたとき](ja/theme/assets.md) — 背景画像・参照形式・検証
- [プラグイン API](ja/plugins.md) — 概要・最小例・子ページへの案内
  - [API リファレンス](ja/plugins/reference.md) — 関数・`ctx`・インターフェース・型・イベントの署名一覧
  - [環境構築と書き方](ja/plugins/setup.md) — 道具の用意・初回の手順・サンプル
  - [実行モデルと寿命](ja/plugins/lifecycle.md) — 隔離・連鎖規則・予算・失敗の数え方
  - [manifest と能力・承認](ja/plugins/manifest.md) — `manifest.json`・能力・承認と状態・設定画面
  - [ビルドと配布・してはいけないこと](ja/plugins/build.md) — 開発手順・禁止事項
