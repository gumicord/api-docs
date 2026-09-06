# manifest と能力・承認

## `manifest.json`

余計なトップレベルキーは不可。`$schema` は
`https://gumicord.dev/schema/plugin-manifest-1.json`。

| フィールド | 必須 | 意味・形 |
|---|---|---|
| `id` | 必須 | 逆ドメイン (`^[a-z0-9]+(\.[a-z0-9_-]+)+$`)。収納ディレクトリ名と一致させる |
| `name` | 必須 | 1〜64 文字。表示名 |
| `version` | 必須 | セマンティック版 (`X.Y.Z`、予告可) |
| `entry` | 任意 | 本体名。平らな `.js` / `.qjsc` のみ、省略時は `plugin.js` |
| `description` | 任意 | 512 文字まで。承認画面に出る説明 |
| `capabilities` | 任意 | 重複なし配列。`"log"`・`"storage"` のみ。未知名は読込失敗 |
| `settings` | 任意 | 平らな `.js` / `.qjsc`。省略時は設定画面なし |

## 能力

宣言したものだけが注入される。書いていない API を呼ぶと `TypeError`
になる (拒否ではなく不在として扱う)。

| 能力 | 与えるもの |
|---|---|
| `log` | `log.info` / `warn` / `error` |
| `storage` | `storage.get` / `set` / `remove` |

`ui.exists` と失敗報告は権限ではなく常時注入される。

`log` 能力の記録:

```ts
info: (msg: string) => void
warn: (msg: string) => void
error: (msg: string) => void
```

効果: 記録に残る。引数は記録文。返す値なし。

`storage` はホスト側の小さな置き場。プラグイン毎に分かれ、再読込を越えて
生きる:

```ts
get: (key: string) => string | null
set: (key: string, value: string) => void
remove: (key: string) => void
getJSON<T>(key: string, fallback: T): T
setJSON(key: string, value: unknown): void
```

- 引数: 鍵と値。`getJSON` の `fallback` は不在時と壊れJSONの両方で返る。
- 返す値: `get` は不在で `null`。他はなし。
- 効果: `set`/`remove` は即時保存される。
- 制限: パッチの中から呼ばない (設定画面の側で保存する)。

## 承認と状態

初見の権限は許可されるまで与えない。記録は
`{"grants": {id: [権限...]}, "disabled": [id...]}` (`grants.json`)。
空列は拒否、キー無しは未確認 (別物)。無効化は許可を残して止めるだけ、
有効化は尋ね直さない。拒否のやり直しと取消は設定画面で行う。

拒否されたプラグインは読み込まない。権限不足のまま動かすと、壊れた
パッチの山になり無効化の勘定を無駄に消費するためである。

## 設定画面

`settings.*` に出す。操作部品はまだ置けない。保存は設定画面の側で行う。

```ts
settings(fn: () => UINode): void
```

表示専用の設定頁を登録する。効果: 設定画面にその頁が出る。

- 引数: ノード工場。呼ばれるのは表示時。
- 返す値: なし。
- 制限: 操作部品は置いても押せない (出来事配送が来たら解禁)。
  保存は設定画面の側で行い、パッチの中から `storage.set` を呼ばない。
