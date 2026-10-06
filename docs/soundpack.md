# サウンドパック（`soundPack`）

通知音を提供します。コードは不要で、`manifest.json` と音声ファイルだけで作れます。`category` は `soundpack` にします。

```json
{
  "manifestVersion": 1,
  "id": "my-sounds",
  "name": "My Sounds",
  "category": "soundpack",
  "soundPack": {
    "sounds": [
      { "id": "ding", "name": "Ding", "file": "ding.m4a" },
      { "id": "chime", "name": "Chime", "file": "sounds/chime.wav" }
    ]
  }
}
```

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `sounds[].id` | ✓ | パック内でのローカル ID。AHQ は `<拡張id>.<id>` に合成して登録する。 |
| `sounds[].name` | ✓ | 設定画面の選択肢に出る表示名。 |
| `sounds[].file` | ✓ | 拡張自身のディレクトリからの相対パス。 |

- 対応する拡張子は `.m4a` / `.mp3` / `.wav` / `.ogg` です。macOS 標準の `.aiff` は使えないので、変換してください（例: `afconvert -f m4af -d aac in.aiff out.m4a`、`afconvert -f WAVE -d LEI16 in.aiff out.wav`）。
- `file` に拡張のディレクトリの外を指すパス（`../` など）は指定できません。
- 音声ファイルの読み込みに失敗すると、そのサウンドパックは登録されません。
- 登録された音は、設定の「完了サウンド」「確認待ちサウンド」で選べます。
- 選択している拡張は、アンインストールできません。
- 展開後の zip の合計サイズは 20MB までです。長い音声は避けてください。
