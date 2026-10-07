# ahq-ext-kit

[English](README.md) | 日本語

[AHQ](https://github.com/ahq-app/ahq) の拡張機能を作るための、仕様書とコーディングエージェント向けの指示（`AGENTS.md`）です。

## 拡張機能で作れるもの

- **テーマ**: 画面とターミナルの配色
- **サウンドパック**: 通知音
- **画面（views）**: AHQ の中に、独自の画面を追加する
- **フック（hooks）**: AHQ の中のイベントに反応する

## はじめかた

コーディングエージェント（Claude Code など）を使って作るのが、一番かんたんです。

```sh
ahq create-extension my-ext   # このキットを my-ext ディレクトリに取得する
cd my-ext
```

その場所でコーディングエージェントを起動し、**作りたい拡張を、そのまま相談してください**。たとえば、次のように頼みます。

- 「Nord に似た、青系の暗いテーマを作って」
- 「完了したときに鳴る、短いチャイムのサウンドパックを作って」
- 「選択中のプロジェクトの名前を表示する画面を作って」

エージェントは、`AGENTS.md` と `docs/en/` の仕様を読み、必要なファイルを作って、AHQ にインストールします。画面付きの拡張など、ビルドが必要なものは、テンプレート（[ahq-ext-template-vite](https://github.com/ahq-app/ahq-ext-template-vite)）を取得して使います。

インストールした拡張は、AHQ を再起動せずに反映されます。見た目や動作を確認して、エージェントに調整を頼んでください。

## 必要なもの

- AHQ と、`ahq` コマンド
- Vite を使う拡張（画面付きなど）を作る場合は、Node.js

## 仕様書

エージェントを使わず、手で作る場合は、次を参照してください。

| ファイル | 内容 |
| --- | --- |
| [docs/ja/manifest.md](docs/ja/manifest.md) | `manifest.json` の共通事項、インストール、zip の作り方 |
| [docs/ja/theme.md](docs/ja/theme.md) | テーマ |
| [docs/ja/soundpack.md](docs/ja/soundpack.md) | サウンドパック |
| [docs/ja/views.md](docs/ja/views.md) | 画面 |
| [docs/ja/hooks.md](docs/ja/hooks.md) | フック |

仕様書は日本語版と英語版があります。日本語版が一次情報で、英語版は日本語版にあわせて更新します（[English](docs/en/)）。
