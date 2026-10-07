# ahq-ext-kit

English | [日本語](README.ja.md)

Specifications and instructions for coding agents (`AGENTS.md`) for building extensions for [AHQ](https://github.com/ahq-app/ahq).

## What you can build with extensions

- **Themes**: colors for the UI and the terminal
- **Sound packs**: notification sounds
- **Screens (views)**: add your own screens inside AHQ
- **Hooks**: react to events inside AHQ

## Getting started

The easiest way is to build with a coding agent (Claude Code, for example).

```sh
ahq create-extension my-ext   # fetches this kit into the my-ext directory
cd my-ext
```

Start your coding agent there and **just tell it what you want to build**. For example:

- "Make a dark, blue-tinted theme similar to Nord"
- "Make a sound pack with a short chime that plays on completion"
- "Make a screen that shows the name of the selected project"

The agent reads the specifications in `AGENTS.md` and `docs/en/`, writes the files that are needed, and installs them into AHQ. For extensions that need a build, such as ones with screens, it fetches and uses the template ([ahq-ext-template-vite](https://github.com/ahq-app/ahq-ext-template-vite)).

An installed extension is applied without restarting AHQ. Check how it looks and behaves, and ask the agent to adjust it.

## Requirements

- AHQ and the `ahq` command
- Node.js, if you build an extension that uses Vite (for example, one with screens)

## Specifications

If you build by hand without an agent, refer to the following.

| File | Content |
| --- | --- |
| [docs/en/manifest.md](docs/en/manifest.md) | Common rules for `manifest.json`, installation, how to make the zip |
| [docs/en/theme.md](docs/en/theme.md) | Themes |
| [docs/en/soundpack.md](docs/en/soundpack.md) | Sound packs |
| [docs/en/views.md](docs/en/views.md) | Screens |
| [docs/en/hooks.md](docs/en/hooks.md) | Hooks |

The specifications are available in English and Japanese ([日本語](docs/ja/)). The Japanese version is the primary source, and the English version is updated to match it.
