# Themes (`theme` / `themes`)

Provides the colors of AHQ's whole UI and the colors of the terminal. No code is needed; you can build one with `manifest.json` alone. Set `category` to `theme`.

## A single theme (`theme`)

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

The theme is registered with the id `<extension id>.theme`.

## Multiple variants (`themes`)

Use this when one theme family has several variants, like Catppuccin's Latte / Mocha.

```json
"themes": [
  { "id": "dark", "name": "My Theme Dark", "tokens": { ... }, "terminalTheme": { ... } },
  { "id": "light", "name": "My Theme Light", "tokens": { ... }, "terminalTheme": { ... } }
]
```

- Each variant is registered with the id `<extension id>.<variant id>.theme`.
- `theme` and `themes` can be written together in the same `manifest.json`.
- Each variant is validated independently. Only a variant whose `tokens` are incomplete is not registered. `ahq extension install` still succeeds and prints no `warning:`. The warning appears in AHQ's screen as an error toast (which disappears after about 3 seconds).

## `tokens` (colors of the UI)

The keys are CSS custom property names, and the values are plain CSS colors (`#rrggbb`, `rgb()`, and CSS functions such as `color-mix()` are all allowed). **All of the following 30 keys are required**; if even one is missing, the theme is not registered.

| Group | Keys |
| --- | --- |
| Accent | `--ahq-color-accent` / `--ahq-color-accent-hover` / `--ahq-color-accent-soft` |
| Background and borders | `--ahq-color-bg` / `--ahq-color-surface-hover` / `--ahq-color-border` / `--ahq-color-border-strong` |
| Text | `--ahq-color-text` / `--ahq-color-text-secondary` / `--ahq-color-text-muted` / `--ahq-color-text-faint` / `--ahq-color-text-inverse` |
| Danger | `--ahq-color-danger` / `--ahq-color-danger-strong` / `--ahq-color-danger-soft` / `--ahq-color-danger-soft-text` |
| Warning | `--ahq-color-warning` / `--ahq-color-warning-soft` / `--ahq-color-warning-soft-text` |
| Success and neutral | `--ahq-color-success-strong` / `--ahq-color-neutral-strong` / `--ahq-color-neutral-strong-text` |
| Worktree | `--ahq-color-worktree-fg` / `--ahq-color-worktree-bg` |
| Sidebar | `--ahq-color-sidebar-bg` / `--ahq-color-sidebar-text` / `--ahq-color-sidebar-border` / `--ahq-color-sidebar-hover` / `--ahq-color-sidebar-button-bg` / `--ahq-color-sidebar-button-hover` |

- Examples of how AHQ itself uses the tokens: `--ahq-color-accent-soft` is used for pale backgrounds, `--ahq-color-success-strong` for the background of toasts, `--ahq-color-neutral-strong` for the background of tooltips, and `--ahq-color-danger-soft-text` for text color.
- You can also derive `--ahq-color-accent-hover` and similar tokens from `--ahq-color-accent` with `color-mix()`.
- For tokens whose purpose is unclear, start with values that follow the same tendency as an existing theme, install the theme into AHQ, and adjust while checking how it actually looks.

## `terminalTheme` (colors of the terminal)

This is passed as is to xterm.js's `ITheme`. Everything is optional, but in practice you should provide the following.

- `background` / `foreground` / `cursor` / `cursorAccent` / `selectionBackground`
- The 16 ANSI colors: `black` / `red` / `green` / `yellow` / `blue` / `magenta` / `cyan` / `white`, and their `bright` versions (`brightBlack` and so on).

**CSS functions such as `color-mix()` cannot be used.** The values are passed to xterm.js without going through the browser's CSS resolution, so use only plain strings such as `#rrggbb` or `rgba(...)`.

## Tips

- Decide the base colors (background, text, accent) first and derive the other tokens from them; the whole theme will feel consistent.
- When you reuse the colors of an existing theme, take the values from that theme's official palette whenever possible. Relying on memory or approximations makes the result drift from the original theme.
- Check the contrast between text and background (especially `--ahq-color-text-secondary` / `--ahq-color-text-muted` / `--ahq-color-text-faint` against the background).
- The extension of the currently selected theme cannot be uninstalled. Switch to another theme before deleting it.
