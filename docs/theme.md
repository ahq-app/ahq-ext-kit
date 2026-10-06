# テーマ（`theme` / `themes`）

AHQ の画面全体の配色と、ターミナルの配色を提供します。コードは不要で、`manifest.json` だけで作れます。`category` は `theme` にします。

## 1 つのテーマ（`theme`）

```json
{
  "manifestVersion": 1,
  "id": "my-theme",
  "name": "My Theme",
  "category": "theme",
  "theme": {
    "tokens": { "--ahq-color-accent": "#cba6f7", "...": "..." },
    "terminalTheme": { "background": "#1e1e2e", "...": "..." }
  }
}
```

テーマは `<拡張id>.theme` という id で登録されます。

## 複数のバリエーション（`themes`）

Catppuccin の Latte / Mocha のように、1 つのテーマファミリーが複数のバリエーションを持つ場合に使います。

```json
"themes": [
  { "id": "dark", "name": "My Theme Dark", "tokens": { ... }, "terminalTheme": { ... } },
  { "id": "light", "name": "My Theme Light", "tokens": { ... }, "terminalTheme": { ... } }
]
```

- 各バリエーションは `<拡張id>.<バリエーションid>.theme` という id で登録されます。
- `theme` と `themes` は同じ `manifest.json` に併記できます。
- バリエーションごとに独立して検証されます。`tokens` が不足しているバリエーションだけが登録されません。`ahq extension install` は成功し、`warning:` も出ません。警告は、AHQ の画面にエラーのトースト（約 3 秒で消える）として表示されます。

## `tokens`（画面の配色）

CSS カスタムプロパティ名をキーに、素の CSS の色を値にします（`#rrggbb`、`rgb()`、`color-mix()` などの CSS 関数も使えます）。**次の 30 個のキーがすべて必要**で、1 つでも欠けるとそのテーマは登録されません。

| グループ | キー |
| --- | --- |
| アクセント | `--ahq-color-accent` / `--ahq-color-accent-hover` / `--ahq-color-accent-soft` |
| 背景・境界 | `--ahq-color-bg` / `--ahq-color-surface-hover` / `--ahq-color-border` / `--ahq-color-border-strong` |
| 文字 | `--ahq-color-text` / `--ahq-color-text-secondary` / `--ahq-color-text-muted` / `--ahq-color-text-faint` / `--ahq-color-text-inverse` |
| 危険 | `--ahq-color-danger` / `--ahq-color-danger-strong` / `--ahq-color-danger-soft` / `--ahq-color-danger-soft-text` |
| 警告 | `--ahq-color-warning` / `--ahq-color-warning-soft` / `--ahq-color-warning-soft-text` |
| 成功・中立 | `--ahq-color-success-strong` / `--ahq-color-neutral-strong` / `--ahq-color-neutral-strong-text` |
| ワークツリー | `--ahq-color-worktree-fg` / `--ahq-color-worktree-bg` |
| サイドバー | `--ahq-color-sidebar-bg` / `--ahq-color-sidebar-text` / `--ahq-color-sidebar-border` / `--ahq-color-sidebar-hover` / `--ahq-color-sidebar-button-bg` / `--ahq-color-sidebar-button-hover` |

- トークンの用途の例（AHQ 本体での使われ方）: `--ahq-color-accent-soft` は淡い背景、`--ahq-color-success-strong` はトーストの背景、`--ahq-color-neutral-strong` はツールチップの背景、`--ahq-color-danger-soft-text` は文字色に使われます。
- `--ahq-color-accent-hover` などは、`--ahq-color-accent` から `color-mix()` で導出する書き方もできます。
- 用途がはっきりしないトークンは、いったん既存のテーマと同じ傾向の値にして、AHQ に入れて実際の見た目を確認しながら調整してください。

## `terminalTheme`（ターミナルの配色）

xterm.js の `ITheme` にそのまま渡されます。すべて任意ですが、実用上は次を揃えます。

- `background` / `foreground` / `cursor` / `cursorAccent` / `selectionBackground`
- ANSI の 16 色: `black` / `red` / `green` / `yellow` / `blue` / `magenta` / `cyan` / `white`、およびそれぞれの `bright` 付き（`brightBlack` など）。

**`color-mix()` などの CSS 関数は使えません。** ブラウザの CSS 解決を経ずに xterm.js に渡るため、`#rrggbb` や `rgba(...)` の素の文字列だけにします。

## 作り方のコツ

- ベースにする色（背景・文字・アクセント）を決めてから、他のトークンを導出すると、全体に統一感が出ます。
- 既存テーマの色を使う場合は、可能な限り、そのテーマの公式パレットから値を取ってください。記憶や近似値に頼ると、元のテーマとずれます。
- 文字と背景のコントラストを確認してください（特に `--ahq-color-text-secondary` / `--ahq-color-text-muted` / `--ahq-color-text-faint` と背景）。
- 選択しているテーマの拡張は、アンインストールできません。切り替えてから削除します。
