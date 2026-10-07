# Developing AHQ extensions (instructions for agents)

You help the user build an extension for AHQ. This directory is where you work. The specifications for extensions are in `docs/en/`. **Always read the relevant specification before writing a file.** Do not guess keys, methods, hook names or permissions that are not in the specification.

Reply to the user in the language they use. (The specifications also exist in Japanese under `docs/ja/`, but you do not need to read them.)

AHQ is a Mac app that runs the sessions of several coding agents side by side. An extension is a directory (or a zip) that contains a `manifest.json`, and you install it into AHQ to use it.

## What to do first

1. Ask the user what they want to build and decide the kind. When unsure, ask the user.

   | What to build | Kind | Files to read | Build |
   | --- | --- | --- | --- |
   | Colors (UI and terminal) | Theme | `docs/en/manifest.md`, `docs/en/theme.md` | Not required |
   | Notification sounds | Sound pack | `docs/en/manifest.md`, `docs/en/soundpack.md` | Not required |
   | A feature with screens (UI) | views | `docs/en/manifest.md`, `docs/en/views.md` | Usually required (Vite) |
   | Reacting to AHQ events | hooks | `docs/en/manifest.md`, `docs/en/hooks.md` | Not required for a single JS file |

2. Decide the extension's `id`. The name of this directory is a candidate for the `id` (it starts with an alphanumeric character, then only alphanumerics, `_` and `-`, up to 64 characters). Ask the user for the `name` (the display name).
3. Check that the `ahq` command is available (`ahq --version`). If it is not, ask the user where the executable inside the AHQ app is (`AHQ.app/Contents/MacOS/ahq`) and run it with that path.

## How to build

### Themes and sound packs (no build)

- Write `manifest.json` (and the audio files, for a sound pack) directly in this directory.
- After writing, always check that you satisfy the required items of the specification. For themes in particular, **all 30 keys of `tokens` are required** (`docs/en/theme.md`). `terminalTheme` cannot use CSS functions such as `color-mix()`.
- Install it.

  ```sh
  zip -r ahq-<category>-<id>-<version>.zip manifest.json <audio files, etc.>
  ahq extension install ahq-<category>-<id>-<version>.zip --overwrite
  ```

  Do not include `docs/` or `AGENTS.md` in the zip.

### Extensions with screens (views)

1. Expand the template into this directory. Files that already exist (such as `README.md`) are not overwritten. The directory name becomes the `id`.

   ```sh
   ahq create-extension . --template vite
   ```

2. Rewrite `manifest.json` (`name`, `description`, `views` and so on) and the screens in `src/ui/` to match the requirements. The template's "My Extension" is written not only in `manifest.json` but also in the `<title>` of `src/ui/index.html` and the `<h1>` of `src/ui/main.ts`, so rewrite those too.
3. Run `npm install`, then `npm run deploy`. It builds, puts the result in a zip and overwrite-installs it into AHQ, all in one run. If `ahq` is not on the PATH, use `AHQ_BIN=<path to the ahq executable> npm run deploy`.
4. Follow `docs/en/views.md` for the restrictions of screens (for example, `type="module"` cannot be used). The template's `vite.config.ts` is configured to satisfy them, so do not change it.

### Extensions with only hooks

- If a single JS file is enough, write `manifest.json` and `main.js` directly, then zip and install them the same way as a theme.
- If you write in TypeScript, build with the template, as for views. Bundle the JS that `main` points to into a **single file**.

## How to check

- An installed extension is applied **without restarting** AHQ. Have the user check the look and behavior in AHQ, and keep adjusting.
- You may overwrite-install as many times as you like until the check is done (`--overwrite`).
- If `ahq extension install` prints a warning (`warning:`), tell the user what it says and fix it.
- Missing `tokens` in a theme do not show up as a `warning:`; they appear only as a toast (which disappears after about 3 seconds) in AHQ's screen. Right after installing, ask the user whether an error toast appeared and whether the theme shows up in the theme list in the settings.
- `ahq extension list` shows whether the extension is listed. On success, `ahq extension install` prints one line. Add `--json` to see what was registered.
- For a sound pack, ask the user to check that `<extension id>.<sound id>` appears under "完了サウンド" (completion sound) and "確認待ちサウンド" (waiting-for-approval sound) in AHQ's settings.

## Rules

- Do not use or suggest features that are not written in the specification. If a needed feature is missing, tell the user that it requires a feature addition on the AHQ side.
- Do not rewrite the contents of `docs/`.
- When you reproduce an existing theme, take the colors from primary sources such as the official palette. Do not rely on memory or approximate values.
- You do not need to raise the extension's `version` on every change, but raise it when you publish.
- Do not put `node_modules/` and `dist/` into git (check the `.gitignore` if there is one).
- Do not delete or overwrite the user's files without confirmation.
