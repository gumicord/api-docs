# 安定 ID カタログ

`select` に書ける ID の一覧。完全一致のみ。ワイルドカード・子孫指定は無い。
追加のみ行われ、削除・改名は行われない。合計 121 個 (中核 77 / 生成可 44)。

- テーマは安定 ID を持つノードにのみ介入できる
- `data` 列があるノードは `ctx.data` の型 ([プラグイン API リファレンス](ja/plugins/reference.md)) と対応する
- 使い方は [テーマ API](ja/theme.md)・[ルール](ja/theme/rules.md) を見ること

## `app.*`

| ID | `data` | 役目 |
|---|---|---|
| `app.root` | — | ツリーの根 |
| `app.window` | — | ウィンドウ 1 枚 |
| `app.screen` | — | 現在表示中の画面を包むコンテナ |
| `app.screen.loading` | — | 起動中 |
| `app.screen.login` | — | ログイン画面 |
| `app.screen.login.title` | — | ログイン画面の見出し |
| `app.screen.login.hint` | — | ログイン画面の説明文・状態表示 |
| `app.screen.login.field` | — | ログインフォームの入力欄 |
| `app.screen.login.label` | — | ログインフォームの欄名 |
| `app.screen.login.error` | — | ログインフォーム上のエラー表示 |
| `app.screen.login.card` | — | ログインフォームのカードコンテナ |
| `app.screen.login.forgot` | — | パスワードを忘れた場合リンク |
| `app.screen.login.divider` | — | または区切り |
| `app.screen.login.qr_button` | — | QRコードログインボタン |
| `app.screen.login.register` | — | アカウント作成リンク |
| `app.screen.main` | — | メイン画面 |

## `chrome.*`

| ID | `data` | 役目 |
|---|---|---|
| `chrome.titlebar` | — | 独自タイトルバー |
| `chrome.titlebar.title` | — | タイトル表示 |
| `chrome.titlebar.controls` | — | ウィンドウ操作ボタン群 |
| `chrome.titlebar.control` | — | 個々のボタン (key で minimize/maximize/close を区別) |

## `nav.*`

| ID | `data` | 役目 |
|---|---|---|
| `nav.guild_list` | — | ギルド一覧 |
| `nav.guild_list.home` | — | DM への入口 |
| `nav.guild_list.item` | `GuildData` | ギルド 1 個 |
| `nav.guild_list.item.icon` | `GuildData` | ギルドアイコン |
| `nav.guild_list.item.pill` | `GuildData` | 左端の白い印。選択中・未読・ホバーで大きさが変わる |
| `nav.guild_list.item.badge` | `GuildData` | 未読・メンション数 |
| `nav.guild_list.folder` | — | サーバフォルダ。押すと開閉する |
| `nav.guild_list.folder.icon` | — | 開いているフォルダの目印 |
| `nav.channel_list` | — | チャンネル一覧 |
| `nav.channel_list.header` | — | ギルド名などの見出し |
| `nav.channel_list.category` | `CategoryData` | カテゴリ |
| `nav.channel_list.item` | `ChannelData` | チャンネル 1 個 |
| `nav.channel_list.item.icon` | `ChannelData` | 種別アイコン |
| `nav.channel_list.item.name` | `ChannelData` | チャンネル名 |
| `nav.channel_list.item.badge` | `ChannelData` | 未読・メンション数 |
| `nav.dm_list` | — | DM 一覧 |
| `nav.dm_list.item` | `DmData` | DM 1 件 |
| `nav.sidebar` | — | 左側全体。一覧と自分をまとめる |
| `nav.sidebar.lists` | — | サーバ一覧とチャンネル一覧 |
| `nav.user_panel` | — | 入っている自分。一覧の下に居座る |
| `nav.user_panel.avatar` | — | 自分のアバター |
| `nav.user_panel.presence` | — | ステータスの点 (key で online/idle/dnd/invisible を区別) |
| `nav.user_panel.name` | — | 自分の表示名 |
| `nav.user_panel.status` | — | ステータスの言葉 |
| `nav.member_list` | — | メンバー一覧 |
| `nav.member_list.sheet` | — | 面の中の一覧。幅を埋める点以外は同じ |
| `nav.member_list.group` | — | 役職やオンラインの見出し |
| `nav.member_list.item` | `MemberData` | メンバー 1 人 |
| `nav.member_list.item.avatar` | `MemberData` | その人のアバター |
| `nav.member_list.item.presence` | `MemberData` | ステータスの点 (key で online/idle/dnd を区別) |
| `nav.member_list.item.name` | `MemberData` | そのサーバでの表示名 |

## `chat.*`

| ID | `data` | 役目 |
|---|---|---|
| `chat.view` | — | チャット領域全体 |
| `chat.header` | `ChannelData` | チャンネルヘッダ |
| `chat.header.title` | `ChannelData` | チャンネル名 |
| `chat.header.topic` | `ChannelData` | トピック |
| `chat.message_list` | — | メッセージ一覧 |
| `chat.message_list.day_divider` | — | 日付の区切り |
| `chat.message` | `MessageData` | メッセージ 1 件 |
| `chat.message.avatar` | `MessageData` | 送信者アイコン |
| `chat.message.header` | `MessageData` | 送信者行 |
| `chat.message.header.author` | `MessageData` | 送信者名 |
| `chat.message.header.badges` | `MessageData` | BOT タグなど |
| `chat.message.header.timestamp` | `MessageData` | 時刻 |
| `chat.message.reply_ref` | `MessageData` | 返信元の参照表示。小アイコンと1行文。押すと元メッセージへ移動する |
| `chat.message.reply_ref.avatar` | — | 参照表示の小アイコン |
| `chat.message.content` | `MessageData` | 本文 |
| `chat.message.content.quote` | — | 引用ブロックの行。中身の高さにだけ合わせる |
| `chat.message.attachments` | `MessageData` | 添付一覧 |
| `chat.message.attachment` | `AttachmentData` | 添付 1 件 |
| `chat.message.embeds` | `MessageData` | 埋め込み一覧 |
| `chat.message.embed` | `EmbedData` | 埋め込み 1 件 |
| `chat.message.actions` | `MessageData` | ホバー時の操作群 |
| `chat.typing_indicator` | — | 入力中表示 |
| `chat.input` | — | 入力欄全体 |
| `chat.input.field` | — | テキスト入力そのもの |
| `chat.input.toolbar` | — | 入力欄の上部 |
| `chat.input.actions` | — | 送信・添付などのボタン群 |

## `overlay.*`

| ID | `data` | 役目 |
|---|---|---|
| `overlay.layer` | — | 浮かせるものを載せる層。開いている間だけ在る |
| `overlay.scrim` | — | 後ろを暗くする覆い |
| `overlay.popover` | — | 基準の点に浮かぶ箱 |
| `overlay.sheet` | — | 下から出てくる面 (携帯) |
| `overlay.sheet.handle` | — | 面の上端の掴みしろ |
| `overlay.drawer` | — | 横から出てくる棚。狭い画面での一覧置き場 |
| `overlay.menu` | — | 操作の並び |
| `overlay.menu.item` | — | 操作 1 つ |
| `overlay.menu.item.icon` | — | 操作の絵 |
| `overlay.menu.item.label` | — | 操作の名前 |
| `overlay.menu.separator` | — | 操作の区切り |
| `overlay.modal` | — | 確かめてから進む窓 |
| `overlay.modal.title` | — | 窓の見出し。何をしようとしているか |
| `overlay.modal.body` | — | 何が起きるかの説明 |
| `overlay.modal.preview` | — | これから起きることの対象そのもの |
| `overlay.modal.actions` | — | 窓のボタン群 |
| `overlay.modal.action` | — | 窓のボタン 1 つ (key で番号を持つ) |
| `overlay.modal.action.label` | — | ボタンの文字 (slot で cancel/confirm/danger) |
| `overlay.tooltip` | — | 指しているものの短い説明。押せず消えるだけ |
| `overlay.toast` | — | 下に出て数秒で消える知らせ。押すものはない |

## `settings.*`

| ID | `data` | 役目 |
|---|---|---|
| `settings.screen` | — | 設定画面 |
| `settings.nav` | — | 設定の分類の並び |
| `settings.page` | — | 開いている分類の中身 |

## `primitive.*`

| ID | `data` | 役目 |
|---|---|---|
| `primitive.text` | — | 文字列 |
| `primitive.image` | — | 画像 |
| `primitive.icon` | — | アイコン |
| `primitive.qr` | — | QR コード |
| `primitive.avatar` | — | 円形の人物画像 |
| `primitive.badge` | — | 小さなラベル |
| `primitive.button` | — | 押せるもの |
| `primitive.divider` | — | 区切り線 |
| `primitive.spinner` | — | 読み込み表示 |
| `primitive.mention` | — | メンション |
| `primitive.emoji` | — | 絵文字 |
| `primitive.code_block` | — | コードブロック |
| `primitive.spoiler` | — | スポイラー |
| `primitive.link` | — | リンク |

## `layout.*`

| ID | `data` | 役目 |
|---|---|---|
| `layout.row` | — | 横並び |
| `layout.column` | — | 縦並び |
| `layout.stack` | — | 重ね |
| `layout.scroll` | — | スクロール領域 |
| `layout.spacer` | — | 空き |
| `layout.scrollbar` | — | スクロール位置の表示と操作 |
| `layout.scrollbar.thumb` | — | スクロールバーの摘み |
