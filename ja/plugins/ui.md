# `ui`

## `ui.patch(id, fn)`

```ts
patch<Id extends NodeId>(id: Id, fn: PatchFn<Id>): void
type PatchFn<Id extends NodeId = NodeId> = (
  node: UINode,
  ctx: PatchContext<Id>,
) => UINode;
```

安定 ID に変形を登録する。効果:

- 走査はボトムアップで、子は適用済みの状態で渡ってくる。
- 照合は適用前の ID に対して行う。出力には再帰しない。
- 同一ノードの複数パッチは登録順に連鎖する。
- 存在しない ID への登録は無害 (走らないだけ)。事前分岐には
  `ui.exists` を使う。
- 仮想化で画面外のノードは訪れない。出来事への反応は Gateway
  イベントミドルウェアで行う。
- `fn` は純粋に書く。1 メッセージに何度呼ばれるかは不定
  (画面外に出て戻るたびに再実行される) ので、カウンタや通信などの
  副作用は予測不能になる。

引数:

- `id`: 安定 ID (`NodeId`)。型で検査され、未知 ID は通らない。
- `fn`: `(node, ctx) => UINode`。返したノードがそのまま使われる。
- `ctx.data`: [データ](ja/plugins/data.md)。ID から型が付く
  (`ctx.data.author.bot` 等)。該当なしは `undefined`。

## `ui.exists(id)`

```ts
exists(id: NodeId): boolean
```

その ID がこの環境に存在しうるか。`chrome.*` はモバイルに無い。
登録自体は存在しない ID でも無害なので、事前分岐のためだけに使う。

- 引数: `id` (安定 ID)。
- 返す値: 存在しうれば `true`。

## `ui.wrap(node, wrapper)`

```ts
wrap(node: UINode, wrapper: Omit<NewUINode, "children">): UINode
```

ノードを子に包んで返す。効果: 元ノードを `children: [node]` の
新しい親の下に置く。

- 引数: `node` (包まれる側)、`wrapper` (親。`children` は渡さない)。
- 返す値: 包んだノード。
- 制限: 包みに core ID (`app.*` / `chrome.*` / `nav.*` / `chat.*`)
  は使えない。自名前空間と `primitive.*` / `layout.*` のみ。

## `ui.after(node, sibling)` / `ui.before(node, sibling)`

```ts
after(node: UINode, sibling: UINode): UINode
before(node: UINode, sibling: UINode): UINode
```

後に／前に兄弟を足す。効果: `{ id: "layout.row", children: [...] }`
で包んで返す。出力は最終形として扱われる。

- 引数: 対象ノードと兄弟ノード。
- 返す値: 包んだ行ノード。

## `ui.stack(nodes)`

```ts
stack(nodes: UINode[]): UINode
```

縦に積む。効果: `{ id: "layout.column", children: nodes }` を返す。

- 引数: 積むノード列。
- 返す値: 縦列ノード。

## `ui.node(id, props?, children?)`

```ts
node(id: CreatableNodeId, props?: Record<string, unknown>, children?: UINode[]): NewUINode
```

自名前空間のノードを作る。効果: `{ id, props?, children? }` を返す。

- 引数: `id` (作れる ID のみ。`plugin.`＋自 ID の `.`→`_`、または
  `primitive.*` / `layout.*`)、`props` (任意の付随値)、`children`。
- 返す値: 新ノード。
- 制限: core ID (`app.*` 等) を作るとその出力ごと捨てられる。

## `ui.text(value)` / `ui.badge(opts)` / `ui.button(opts)` / `ui.icon(name)`

```ts
text(value: string): NewUINode
badge(opts: { text: string; tone?: string }): NewUINode
button(opts: { label: string; onPress: () => void }): NewUINode
icon(name: string): NewUINode
```

`primitive.text` / `primitive.badge` / `primitive.button` /
`primitive.icon` を作る。効果: 対応する `id` と `props` を持つ
ノードを返す。

- 引数: 表示文字列・標識・標識＋押下動作・絵文字名。
- 返す値: 新ノード。
- 注意: `onPress` 等の関数はホスト境界を越えられず黙って落とす。
  `tone` 等の見た目指定もテーマの持ち物のため落とす。`button` は
  設定画面では押せない。
