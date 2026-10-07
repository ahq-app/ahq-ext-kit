# Hooks (`hooks`)

Lets the extension's JS react to events that happen inside AHQ. Set `category` to `addon`.

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

| Field | Required | Description |
| --- | --- | --- |
| `hooks` | - | An array of the names of the hooks you want to react to. Only the names in the list below can be written. An unknown name causes an error at install time. |
| `main` | ✓ when `hooks` is present | The extension's JS file. A path relative to the extension's own directory, and it must end with `.js`. |

## Available hooks

| Name | Kind | Fires when | Argument |
| --- | --- | --- | --- |
| `taskUpdated` | Notification only | The content of a task (a request to Claude Code) changes | The task as of the change |
| `terminalTabUpdated` | Notification only | The content of a terminal tab (a tab that opens a shell without starting Claude) changes | The terminal tab as of the change |

`taskUpdated` passes only tasks, that is, requests to Claude Code. Receive terminal tabs with `terminalTabUpdated` (the former `taskUpdated` also passed terminal tabs).

## How to write it

In the JS of `main`, **only the names of the hooks you declared** are visible, as `ahq.hooks.<name>(handler)`. Names you did not declare are not visible at all.

```js
ahq.hooks.taskUpdated(function (task) {
  console.log('task updated: ' + task.Title + ' (' + task.Status + ')')
})
```

- The return value of a handler is ignored.
- An exception in a handler only appears in AHQ's log; it does not affect AHQ's behavior.

### Argument of `taskUpdated` (task)

**The property names start with an uppercase letter, exactly as the field names of the Go struct** (`ID`, not `id`). The main ones:

| Property | Description |
| --- | --- |
| `ID` | The task's ID |
| `Title` | The title |
| `Status` | One of `composing` / `running` / `waiting_approval` / `idle` / `archived` |
| `Kind` | Currently `agent` (it can also be empty). Earlier versions also put `terminal` / `markdown` here, but those are now handled as separate kinds of tabs |
| `AttentionKind` | The kind of thing that needs the user's attention: `""` (none) / `question` / `completed` |
| `LastPreview` | A preview of the latest output |
| `ActiveSessionID` | The ID of the active session |

### Argument of `terminalTabUpdated` (terminal tab)

As with tasks, the property names start with an uppercase letter. The main ones:

| Property | Description |
| --- | --- |
| `ID` | The terminal tab's ID |
| `Title` | The title (the name of the command in the foreground of the shell) |
| `Status` | One of `idle` / `running` / `archived` |
| `ActiveSessionID` | The ID of the active session |
| `WorktreePath` | The path of the working directory's worktree (empty if there is none) |
| `SourceTaskID` | The ID of the task that opened this tab (empty if there is none) |

These shapes may change in the future (for example, to names starting with a lowercase letter).

## Runtime

The JS of `main` runs in the JavaScript runtime built into AHQ (goja).

- **There are no browser APIs (`document`, `fetch`, etc.) and no Node APIs (`require`, `fs`, etc.).** All you can use is standard JavaScript, `ahq` and `console`.
- `console.log` / `console.warn` / `console.error` are available. The output goes not to a screen but to the log of the AHQ process, in the form `extension <id> console.<level>: ...` (in the terminal, if you started it with `wails dev` or from a terminal).
- `main` is a **single file**. Other files cannot be loaded with `import` or `require`, so if you write in TypeScript or in several files, bundle them into one JS file.

## Installing and applying

- When an extension is installed (overwritten) or uninstalled, the hooks are recreated (or discarded) without restarting AHQ.
- If loading the JS of `main` fails (for example, a syntax error), the extension's hooks are disabled. A warning is shown when you run `ahq extension install`.
