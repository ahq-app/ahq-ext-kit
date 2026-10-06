# 画面（`views`）

拡張に画面（UI）を持たせます。画面は HTML / CSS / JS で作ります。TypeScript や SCSS を使う場合は、ビルドしてから同梱します。`category` は `addon` にします。

```json
{
  "manifestVersion": 1,
  "id": "my-extension",
  "name": "My Extension",
  "category": "addon",
  "version": "0.1.0",
  "views": [
    {
      "id": "main",
      "title": "My Extension",
      "location": "peek",
      "entry": "ui/index.html",
      "icon": "ui/icon.svg",
      "permissions": ["project.getCurrent"]
    }
  ]
}
```

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `id` | ✓ | 拡張内でのローカル ID。AHQ は `<拡張id>.<id>` に合成する。 |
| `title` | ✓ | 画面のタイトル。`icon` が無いときは、呼び出しボタンの文字にもなる。 |
| `location` | ✓ | 画面を表示する場所（下記）。 |
| `entry` | ✓ | 画面の HTML ファイル。拡張自身のディレクトリからの相対パス。 |
| `icon` | - | 呼び出しボタンに使う画像ファイル（拡張のディレクトリからの相対パス）。省略すると `title` が文字で表示される。 |
| `permissions` | - | この画面が AHQ 本体に呼び出せるメソッドの許可リスト（下記）。 |

## 表示する場所（`location`）

| 値 | 表示 |
| --- | --- |
| `peek` | 右側にスライドして開くパネル。ツールバーに、拡張ごとのボタンが追加される。同時に開けるのは 1 つ。 |
| `settings` | 設定画面に、この拡張のセクションが追加され、固定の高さ（600px 程度）の領域に埋め込まれる。収まらない分は、画面の内側でスクロールする。 |

どちらも、画面の高さを AHQ に伝える仕組みはありません。領域いっぱいに表示される前提で作ってください。

## `entry` と同梱するファイル

- `entry` には、**ビルド済み**の HTML を指定します。`.svelte` などの未コンパイルのソースは使えません。
- HTML から、CSS・JS・画像を**相対パス**で参照できます。
- 画面のファイルは、AHQ が拡張のディレクトリから毎回読み込みます。拡張を上書きインストールすると、開いている `peek` の画面は読み込み直されます。

## 実行環境の制約

画面は、`<iframe sandbox="allow-scripts">` で表示されます（`allow-same-origin` は付きません）。そのため、次の制約があります。

- AHQ 本体の DOM・Cookie・localStorage に、一切アクセスできません。画面は、毎回別のオリジンとして扱われます。
- **`<script type="module">` は読み込めません。** ブラウザが CORS でブロックするためです。JS は、通常の `<script src="...">` か、インラインのスクリプトで読み込みます。TypeScript などを使う場合は、**1 つの IIFE にまとめて出力**します（`vite.config.ts` の `build.lib` で `formats: ['iife']`）。
- AHQ 本体の機能は、`window.ahq.call()` 経由でだけ使えます（下記）。

## AHQ が自動で注入するもの

画面の HTML を配信するとき、AHQ が `</head>` の直前に `<script src="/ext-runtime.js"></script>` を挿入します。拡張側の HTML に書く必要はありません。これにより、次が使えます。

### `window.ahq.call(method, params)`

AHQ 本体のメソッドを呼びます。`Promise` を返し、失敗すると `Error` で reject されます。

```ts
const project = await window.ahq.call('project.getCurrent')
```

- 呼べるのは、`manifest.json` の `permissions` に宣言したメソッドだけです。宣言していないメソッドは、AHQ 側で拒否されます。
- `permissions` に、下の一覧に無い名前を書くと、インストール時にエラーになります。

### テーマへの追従

- AHQ は、現在のテーマの色（`--ahq-color-*`、一覧は [theme.md](theme.md)）を、画面の `<html>` の CSS カスタムプロパティとして設定し、テーマが変わるたびに更新します。
- CSS で `var(--ahq-color-text)` のように使うと、AHQ のテーマに追従します。
- 何も書かなければ、背景色と文字色が AHQ のテーマになる、最小限の既定のスタイルが適用されます。拡張側の CSS は、これより優先されます。

## 呼べるメソッドの一覧

| メソッド | パラメータ | 戻り値 | 説明 |
| --- | --- | --- | --- |
| `project.getCurrent` | なし | `{ id, name, path }` または `null` | この画面を開いた時点で選択していたプロジェクト。画面を開いている間に別のプロジェクトへ切り替えても、変わらない。 |
| `task.openComposerDraft` | `{ text: string }` | `null` | 現在のプロジェクトに、新しいタスクの下書きを開き、`text` を入力済みにする（送信は、ユーザーが確定する）。プロジェクトが選択されていないとエラー。 |
| `github.listIssues` | なし | `{ number, title, state, labels, updatedAt, commentCount }[]` | 現在のプロジェクトの GitHub Issue の一覧。 |
| `github.getIssue` | `{ number }` | `{ number, title, body, state, labels, author, comments: { author, body, createdAt }[] }` | Issue の詳細。 |
| `github.commentIssue` | `{ number, body }` | `null` | Issue にコメントする。 |
| `github.closeIssue` | `{ number }` | `null` | Issue を閉じる。 |
| `github.getIssueTaskTemplate` | なし | `string` | Issue からタスクを作るときのテンプレート（保存済みの値）。 |
| `github.setIssueTaskTemplate` | `{ template: string }` | `null` | 同テンプレートを保存する。 |
| `github.getDefaultIssueTaskTemplate` | なし | `string` | 同テンプレートの既定値。 |

- `github.*` は、ローカルの `gh` CLI を使い、現在のプロジェクトのディレクトリで実行されます。AHQ が GitHub のトークンを保存することはありません。
- `github.*` の権限を要求する画面は、選択中のプロジェクトが GitHub のリポジトリでないとき、ボタンが無効になります。
- 上記以外のメソッドは、現時点ではありません。必要な機能がある場合は、AHQ の Issue で相談してください。

## 開発の進め方

`ahq create-extension . --template vite` で、Vite + TypeScript + SCSS のテンプレートを現在のディレクトリに展開できます。

- `npm run deploy`: ビルドして zip にまとめ、AHQ へ上書きインストールします（1 回で終了します）。
- `npm start`: ウォッチしながらビルドし、保存のたびにインストールします。
- アイコンは Iconify が使えます（`unplugin-icons`）。
- テンプレートは、画面の JS を 1 つの IIFE として出力するようにあらかじめ設定されています。

ビルドを使わず、素の HTML / CSS / JS だけで作る場合は、`manifest.json` と画面のファイルを zip にまとめるだけで足ります。
