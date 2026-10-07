# Screens (`views`)

Lets an extension have its own screens (UI). Screens are built with HTML / CSS / JS. If you use TypeScript or SCSS, build first and bundle the result. Set `category` to `addon`.

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

| Field | Required | Description |
| --- | --- | --- |
| `id` | ✓ | A local ID within the extension. AHQ composes it into `<extension id>.<id>`. |
| `title` | ✓ | The title of the screen. When there is no `icon`, it is also used as the text of the button that opens the screen. |
| `location` | ✓ | Where the screen is shown (see below). |
| `entry` | ✓ | The HTML file of the screen, as a path relative to the extension's own directory. |
| `icon` | - | An image file for the button that opens the screen (a path relative to the extension's directory). If omitted, `title` is shown as text. |
| `permissions` | - | An allowlist of the methods this screen may call on AHQ itself (see below). Can be omitted if the screen calls no methods. |

## Where the screen is shown (`location`)

| Value | Display |
| --- | --- |
| `peek` | A panel that slides open on the right. A button for each extension is added to the toolbar. Only one can be open at a time. |
| `settings` | A section for this extension is added to the settings screen, and the screen is embedded in an area of fixed height (about 600px). Content that does not fit scrolls inside the screen. |

In both cases there is no mechanism for the screen to tell AHQ its height. Build the screen on the assumption that it fills the area.

## `entry` and bundled files

- Specify a **built** HTML file in `entry`. Uncompiled sources such as `.svelte` cannot be used.
- The HTML can refer to CSS, JS and images by **relative paths**.
- AHQ reads the screen's files from the extension's directory every time. When you overwrite-install the extension, an open `peek` screen is reloaded.

## Restrictions of the runtime

Screens are displayed in `<iframe sandbox="allow-scripts">` (without `allow-same-origin`). This brings the following restrictions.

- A screen cannot access AHQ's DOM, cookies or localStorage at all. Each time, the screen is treated as a different origin.
- **`<script type="module">` cannot be loaded.** The browser blocks it because of CORS. Load JS with a normal `<script src="...">` or an inline script. If you use TypeScript or the like, **output everything as a single IIFE** (`formats: ['iife']` in `build.lib` of `vite.config.ts`).
- AHQ's features are available only through `window.ahq.call()` (see below).

## What AHQ injects automatically

When AHQ serves the HTML of a screen, it inserts `<script src="/ext-runtime.js"></script>` just before `</head>`. You do not need to write it in the extension's HTML. This makes the following available.

### `window.ahq.call(method, params)`

Calls a method of AHQ itself. It returns a `Promise`, which is rejected with an `Error` on failure.

```ts
const project = await window.ahq.call('project.getCurrent')
```

- You can call only the methods declared in `permissions` of `manifest.json`. Methods that are not declared are rejected by AHQ.
- If you write a name in `permissions` that is not in the list below, installation fails with an error.

### Following the theme

- AHQ sets the colors of the current theme (`--ahq-color-*`; the list is in [theme.md](theme.md)) as CSS custom properties on the screen's `<html>`, and updates them every time the theme changes.
- If you use them in CSS like `var(--ahq-color-text)`, your screen follows AHQ's theme.
- If you write nothing, a minimal default style is applied in which the background and text colors come from AHQ's theme. The extension's own CSS takes precedence over it.

## Callable methods

| Method | Parameters | Return value | Description |
| --- | --- | --- | --- |
| `project.getCurrent` | None | `{ id, name, path }` or `null` | The project that was selected when this screen was opened. It does not change even if the user switches to another project while the screen is open. |
| `task.openComposerDraft` | `{ text: string }` | `null` | Opens a new task draft for the current project with `text` already entered (the user confirms the submission). It is an error if no project is selected. |
| `github.listIssues` | None | `{ number, title, state, labels, updatedAt, commentCount }[]` | The list of GitHub Issues of the current project. |
| `github.getIssue` | `{ number }` | `{ number, title, body, state, labels, author, comments: { author, body, createdAt }[] }` | The details of an Issue. |
| `github.commentIssue` | `{ number, body }` | `null` | Comments on an Issue. |
| `github.closeIssue` | `{ number }` | `null` | Closes an Issue. |
| `github.getIssueTaskTemplate` | None | `string` | The template used when creating a task from an Issue (the saved value). |
| `github.setIssueTaskTemplate` | `{ template: string }` | `null` | Saves that template. |
| `github.getDefaultIssueTaskTemplate` | None | `string` | The default value of that template. |

- `github.*` uses the local `gh` CLI and runs in the current project's directory. AHQ never stores a GitHub token.
- A screen that requests `github.*` permissions has its button disabled when the selected project is not a GitHub repository.
- There are no methods other than the above at the moment. If you need a feature, discuss it in an AHQ Issue.

## How to develop

`ahq create-extension . --template vite` expands a Vite + TypeScript + SCSS template into the current directory.

- `npm run deploy`: builds, puts the result in a zip and overwrite-installs it into AHQ (it finishes in one run).
- `npm start`: builds while watching and installs on every save.
- `npm run check`: type-checks (`tsc --noEmit`).
- If `ahq` is not on your PATH, specify the executable with `AHQ_BIN` (e.g. `AHQ_BIN="/path/to/ahq.app/Contents/MacOS/ahq" npm run deploy`).
- Icons can come from Iconify (`unplugin-icons`).
- The template is preconfigured to output the screen's JS as a single IIFE.

If you build with plain HTML / CSS / JS and no build step, it is enough to put `manifest.json` and the screen's files in a zip.
