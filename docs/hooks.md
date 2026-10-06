# フック（`hooks`）

AHQ の中で起きるイベントに、拡張の JS が反応できるようにします。`category` は `addon` にします。

```json
{
  "manifestVersion": 1,
  "id": "my-extension",
  "name": "My Extension",
  "category": "addon",
  "hooks": ["taskUpdated"],
  "main": "main.js"
}
```

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `hooks` | - | 反応したいフックの名前の配列。下の一覧にある名前だけが書ける。未知の名前はインストール時にエラーになる。 |
| `main` | `hooks` があるとき ✓ | 拡張の JS ファイル。拡張自身のディレクトリからの相対パスで、`.js` で終わる必要がある。 |

## 使えるフック

| 名前 | 種類 | 発火するとき | 引数 |
| --- | --- | --- | --- |
| `taskUpdated` | 通知のみ | タスクの内容が変わったとき | 変わった時点のタスク |

## 書き方

`main` の JS から、**宣言したフックの名前だけが** `ahq.hooks.<名前>(ハンドラ)` として見えます。宣言していない名前は、存在自体が見えません。

```js
ahq.hooks.taskUpdated(function (task) {
  console.log('task updated: ' + task.Title + ' (' + task.Status + ')')
})
```

- ハンドラの戻り値は無視されます。
- ハンドラ内の例外は、AHQ のログに出るだけで、AHQ の動作には影響しません。

### `taskUpdated` の引数（タスク）

**プロパティ名は、Go の構造体のフィールド名どおりの大文字始まりです**（`id` ではなく `ID`）。主なもの:

| プロパティ | 説明 |
| --- | --- |
| `ID` | タスクの ID |
| `Title` | タイトル |
| `Status` | `composing` / `running` / `waiting_approval` / `idle` / `archived` のいずれか |
| `Kind` | `agent` / `terminal` / `markdown` のいずれか（空のこともある） |
| `AttentionKind` | ユーザーの注意を要する種類。`""`（なし）/ `question` / `completed` |
| `LastPreview` | 直近の出力のプレビュー |
| `ActiveSessionID` | アクティブなセッションの ID |

この形は、将来変わる可能性があります（小文字始まりへの変更など）。

## 実行環境

`main` の JS は、AHQ の中に組み込まれた JavaScript の実行環境（goja）で動きます。

- **ブラウザの API（`document`・`fetch` など）や、Node の API（`require`・`fs` など）はありません。** 使えるのは、標準の JavaScript と、`ahq`・`console` だけです。
- `console.log` / `console.warn` / `console.error` が使えます。出力は、画面ではなく AHQ のプロセスのログに、`extension <id> console.<level>: ...` の形式で出ます（`wails dev` や、ターミナルから起動した場合は、そのターミナル）。
- `main` は**単一のファイル**です。`import` や `require` で他のファイルを読み込めないため、TypeScript や複数ファイルで書く場合は、1 つの JS にバンドルしてください。

## インストールと反映

- 拡張を（上書き）インストール、またはアンインストールすると、AHQ を再起動せずに、フックが作り直される（または破棄される）。
- `main` の JS の読み込みに失敗すると（構文エラーなど）、その拡張のフックは無効になります。`ahq extension install` の実行時に、警告として表示されます。
