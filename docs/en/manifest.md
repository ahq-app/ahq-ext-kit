# manifest.json and common rules for extensions

An AHQ extension is a directory containing a `manifest.json` (or a zip of that directory). When you install it into AHQ, it is placed under `extensions/<id>/` in AHQ's data area.

## Basic structure

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

| Field | Required | Description |
| --- | --- | --- |
| `manifestVersion` | ✓ | `1` (an integer of 1 or more). |
| `id` | ✓ | The extension's global ID. Starts with an alphanumeric character, then only alphanumerics, `_` and `-`, up to 64 characters in total (`^[a-zA-Z0-9][a-zA-Z0-9_-]{0,63}$`). |
| `name` | ✓ | Display name. Shown in the extension list. |
| `category` | ✓ | Classification in the list. One of `theme` / `soundpack` / `addon`. |
| `version` | - | The extension's own version (e.g. `1.2.0`). Any format is accepted and shown as is. |
| `author` / `authorUrl` / `homepage` | - | The author's name, the author's URL, and the extension's home page. |
| `description` | - | A short description shown in the list. |
| `icon` | - | An image file, as a path relative to the extension's own directory (e.g. `icon.png`). Shown in the list. If omitted, a default icon is used. |
| `readme` / `license` | - | File names of the bundled README / LICENSE. Even if omitted, files with conventional names such as `README`, `README.md`, `LICENSE` and `LICENSE.txt` are found automatically. |

### Choosing a `category`
- `theme`: an extension that provides only themes.
- `soundpack`: an extension that provides only sound packs.
- `addon`: everything else (extensions with `hooks` or `views`).

## Contributions

You declare what an extension provides with the following keys. **At least one is required**, and you can combine several. Unknown keys are ignored.

| Key | What it provides | Runs code | Details |
| --- | --- | --- | --- |
| `theme` | A color theme (one) | No | [theme.md](theme.md) |
| `themes` | Color themes (several variants) | No | [theme.md](theme.md) |
| `soundPack` | Notification sounds | No | [soundpack.md](soundpack.md) |
| `views` | Screens (UI) | Yes (inside an iframe) | [views.md](views.md) |
| `hooks` + `main` | Reactions to AHQ events | Yes (in AHQ's built-in JS runtime) | [hooks.md](hooks.md) |

Themes and sound packs are data-only extensions without code. They can consist of just `manifest.json` and, if needed, audio files. No build is required.

## Classification by how you build it

| What you want to build | Build | Notes |
| --- | --- | --- |
| Theme / sound pack | Not required | Just write `manifest.json` (and audio files) and put them in a zip. |
| Extension with screens | Usually required | If you use TypeScript / SCSS, you need to build with Vite. A template is available ([views.md](views.md)). |
| Hooks-only extension | Not required for a single JS file | A build is needed if you write it in TypeScript. |

## Installing and managing

Put the extension in a zip and install it with the `ahq` command. If AHQ is running, the change is applied without restarting (themes, sounds, screens and hooks alike).

```sh
ahq extension install <path or URL of the zip> [--overwrite] [--json]
ahq extension list
ahq extension uninstall <id>
```

- On success, it prints one line: `installed <id> (<name> v<version>)`. With `--json`, it prints the registered content as JSON (including `warnings`; the `dataUrl` of audio and similar is excluded). Use the exit code to tell whether it succeeded.
- If an extension with the same `id` already exists, it is an error unless you pass `--overwrite`. Always pass `--overwrite` during development.
- An extension that is currently selected as the theme or sound cannot be uninstalled.
- If `ahq` is not on your PATH, specify the executable inside the AHQ app directly (`AHQ.app/Contents/MacOS/ahq`).

### Making the zip
- Put `manifest.json` **directly at the root** of the zip (or wrapped in exactly one directory level).
- The total size after extraction is limited to 20MB.
- The name of the install directory is decided by the `id` in `manifest.json`, not by the structure of the zip.
- Name the file `ahq-<category>-<id>-<version>.zip` (e.g. `ahq-addon-my-extension-0.1.0.zip`).

```sh
# Example of an extension that needs no build (theme / sound pack)
zip -r ahq-theme-my-theme-0.1.0.zip manifest.json   # add audio files too, if any
ahq extension install ahq-theme-my-theme-0.1.0.zip --overwrite
```

## Validation

`manifest.json` is validated at install time, and a problem causes an error. The main causes are:

- `manifestVersion` / `name` / `id` / `category` is missing or invalid.
- `category` is none of `theme` / `soundpack` / `addon`.
- When you place a directory by hand, its `id` does not match the directory name.
- There is no contribution at all (`theme` / `themes` / `soundPack` / `hooks` / `views`).
- A required item of a contribution is missing (see each document for details).

## Naming rules for ids

The ids of AHQ's built-in themes and sounds do not contain `.`. What an extension provides is composed by AHQ into a dot-separated form (`<extension id>.theme`, `<extension id>.<variant id>.theme`, `<extension id>.<sound id>`, `<extension id>.<view id>`). Because of this, they never collide with built-in ids. The extension itself does not need to put a `.` in its ids.
