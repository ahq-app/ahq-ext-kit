# Sound packs (`soundPack`)

Provides notification sounds. No code is needed; you can build one with `manifest.json` and audio files alone. Set `category` to `soundpack`.

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

| Field | Required | Description |
| --- | --- | --- |
| `sounds[].id` | ✓ | A local ID within the pack. AHQ composes it into `<extension id>.<id>` when registering. |
| `sounds[].name` | ✓ | The display name shown in the options on the settings screen. |
| `sounds[].file` | ✓ | A path relative to the extension's own directory. |

- The supported extensions are `.m4a` / `.mp3` / `.wav` / `.ogg`. The macOS-standard `.aiff` cannot be used, so convert it (e.g. `afconvert -f m4af -d aac in.aiff out.m4a`, `afconvert -f WAVE -d LEI16 in.aiff out.wav`).
- `file` cannot point outside the extension's directory (e.g. with `../`).
- If an audio file fails to load, the whole sound pack is not registered.
- Registered sounds can be chosen in the settings under "完了サウンド" (completion sound) and "確認待ちサウンド" (waiting-for-approval sound).
- The extension that is currently selected cannot be uninstalled.
- The total size of the extracted zip is limited to 20MB. Avoid long audio.
