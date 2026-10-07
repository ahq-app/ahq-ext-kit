# manifest.json と拡張機能の共通事項

AHQ の拡張機能は、`manifest.json` を持つディレクトリ（またはそれを固めた zip）です。AHQ にインストールすると、AHQ のデータ領域の `extensions/<id>/` に置かれます。

## 基本の構造

```json
{
  "manifestVersion": 1,
  "id": "my-extension",
  "name": "My Extension",
  "category": "addon",
  "version": "0.1.0",
  "views": [ ... ]
}
```

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `manifestVersion` | ✓ | `1`（1 以上の整数）。 |
| `id` | ✓ | 拡張のグローバル ID。英数字で始まり、英数字・`_`・`-` のみ、64 文字以内（`^[a-zA-Z0-9][a-zA-Z0-9_-]{0,63}$`）。 |
| `name` | ✓ | 表示名。拡張機能の一覧に出る。 |
| `category` | ✓ | 一覧での分類。`theme` / `soundpack` / `addon` のいずれか。 |
| `version` | - | 拡張自身のバージョン（例: `1.2.0`）。形式は自由で、そのまま表示される。 |
| `author` / `authorUrl` / `homepage` | - | 作者名・作者の URL・拡張のホームページ。 |
| `description` | - | 一覧に出る短い説明。 |
| `icon` | - | 拡張自身のディレクトリからの相対パスで指す画像ファイル（例: `icon.png`）。一覧に出る。省略すると既定のアイコンになる。 |
| `readme` / `license` | - | 同梱する README / LICENSE のファイル名。省略しても、`README`・`README.md`・`LICENSE`・`LICENSE.txt` などの慣習的な名前のファイルは自動で見つかる。 |

### `category` の選び方
- `theme`: テーマだけを提供する拡張。
- `soundpack`: サウンドパックだけを提供する拡張。
- `addon`: それ以外（`hooks` や `views` を持つ拡張）。

## 提供機能（contribution）

拡張が何を提供するかは、次のキーで宣言します。**少なくとも 1 つは必要**で、複数を組み合わせることもできます。未知のキーは無視されます。

| キー | 提供するもの | コードの実行 | 詳細 |
| --- | --- | --- | --- |
| `theme` | 配色テーマ（1 つ） | なし | [theme.md](theme.md) |
| `themes` | 配色テーマ（複数のバリエーション） | なし | [theme.md](theme.md) |
| `soundPack` | 通知音 | なし | [soundpack.md](soundpack.md) |
| `views` | 画面（UI） | あり（iframe 内） | [views.md](views.md) |
| `hooks` + `main` | AHQ のイベントへの反応 | あり（AHQ 内の JS 実行環境） | [hooks.md](hooks.md) |

テーマとサウンドパックは、コードを持たないデータだけの拡張です。`manifest.json` と、必要なら音声ファイルだけで構成できます。ビルドも不要です。

## 作り方による分類

| 作りたいもの | ビルド | 備考 |
| --- | --- | --- |
| テーマ・サウンドパック | 不要 | `manifest.json`（と音声ファイル）を書いて zip にまとめるだけ。 |
| 画面付きの拡張 | 通常は必要 | TypeScript / SCSS を使うなら Vite でのビルドが必要。テンプレートがある（[views.md](views.md)）。 |
| hooks だけの拡張 | JS 1 ファイルなら不要 | TypeScript で書くならビルドが必要。 |

## インストールと管理

拡張は、zip にまとめて `ahq` コマンドでインストールします。AHQ が起動中なら、再起動せずに反映されます（テーマ・サウンド・画面・フックのすべて）。

```sh
ahq extension install <zip のパス または URL> [--overwrite] [--json]
ahq extension list
ahq extension uninstall <id>
```

- 成功すると `installed <id> (<name> v<version>)` の 1 行を出力します。`--json` を付けると、登録された内容（`warnings` を含む。音声などの `dataUrl` は除く）を JSON で出力します。成否は終了コードで判断します。
- 同じ `id` の拡張が既にあると、`--overwrite` を付けない限りエラーになります。開発中は常に `--overwrite` を付けます。
- 今テーマ・サウンドとして選択されている拡張は、アンインストールできません。
- AHQ が `ahq` を PATH に持たない場合は、AHQ アプリ内の実行ファイル（`AHQ.app/Contents/MacOS/ahq`）を直接指定します。

### zip の作り方
- `manifest.json` が zip の**ルート直下**にある形（または 1 階層だけラップした形）にします。
- 展開後の合計サイズは 20MB までです。
- インストール先のディレクトリ名は、zip の構造ではなく、`manifest.json` の `id` で決まります。
- ファイル名は、`ahq-<category>-<id>-<version>.zip`（例: `ahq-addon-my-extension-0.1.0.zip`）にします。

```sh
# ビルド不要の拡張（テーマ・サウンドパック）の例
zip -r ahq-theme-my-theme-0.1.0.zip manifest.json   # 音声ファイルがあれば一緒に
ahq extension install ahq-theme-my-theme-0.1.0.zip --overwrite
```

## 検証

インストール時に `manifest.json` が検証され、問題があるとエラーになります。主な原因は次のとおりです。

- `manifestVersion` / `name` / `id` / `category` が無い、または不正。
- `category` が `theme` / `soundpack` / `addon` のいずれでもない。
- 手動でディレクトリに置いた場合に、`id` がディレクトリ名と一致しない。
- 提供機能（`theme` / `themes` / `soundPack` / `hooks` / `views`）が 1 つも無い。
- 各提供機能の必須項目の不足（詳細は各ドキュメント）。

## id の命名規則

AHQ 本体のテーマ・サウンドの id は `.` を含みません。拡張が提供するものは、AHQ が `.` で区切った形に合成します（`<拡張id>.theme`、`<拡張id>.<バリエーションid>.theme`、`<拡張id>.<サウンドid>`、`<拡張id>.<viewのid>`）。そのため、本体の id と衝突することはありません。拡張自身が、id に `.` を含める必要はありません。
