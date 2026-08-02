# gstack-installer (retired)

> [!IMPORTANT]
> **This project is retired and the repo is archived.** Install gstack with the `gstack` plugin instead:
> `/plugin marketplace add smk-labs/claude-plugins` then `/plugin install gstack@smk`.
>
> Two ways to install one thing was one too many. The `gstack` plugin is a self-contained snapshot that works out of the box; this installer's only advantage was tracking upstream, which is not worth a second repo to maintain. For the real native setup, run Garry Tan's own installer from [garrytan/gstack](https://github.com/garrytan/gstack).

<details>
<summary>Original README</summary>

# gstack-installer

A one-plugin Claude Code marketplace that bootstraps [**gstack**](https://github.com/garrytan/gstack), Garry Tan's Claude Code setup (70+ slash-command skills: CEO/eng/design review, QA, browser automation, ship/release, security audits).

## Why this exists (read first)

gstack is **not** a Claude Code plugin. It is a skills bundle that installs to `~/.claude/skills/gstack` through its own `./setup` build script. Three reasons it cannot be loaded by `/plugin install` directly:

- Its skills live at the **repo root** (`review/`, `qa/`, `ship/`, ...), not under `skills/`, which is the only place the plugin loader scans.
- Every skill **hardcodes `~/.claude/skills/gstack/bin/...`** paths that only exist after `./setup` runs.
- Its `/browse` and `/qa` engine is a **compiled binary** built by Bun during `./setup`. Plugin install runs no build step.

So pointing a marketplace's plugin `source` at `garrytan/gstack` would install nothing usable. This marketplace instead ships a single **installer plugin**: you add the marketplace, install the plugin, and run one skill that performs gstack's real upstream clone + setup. No fork, no vendored copy, always current.

## Install

### Claude Desktop

1. Settings -> Extensions (Plugins) -> **Add marketplace**, paste the **full GitHub URL** (Desktop requires the full address, not the `owner/repo` shorthand): `https://github.com/smk-labs/gstack-installer`
2. Find **gstack-installer** in the marketplace list, click **Install**.
3. In a chat, run `/gstack-installer:install-gstack` (or just type "install gstack").
4. Restart Claude Desktop so the gstack skills load.

### Claude Code (CLI)

```bash
/plugin marketplace add smk-labs/gstack-installer
/plugin install gstack-installer@gstack
/gstack-installer:install-gstack
```

Then restart Claude Code.

### Test it locally before publishing

```bash
/plugin marketplace add ~/gstack-installer
/plugin install gstack-installer@gstack
```

## What the installer skill does

1. Checks prerequisites (`git`, `bun`; `node` on Windows).
2. Clones gstack to `~/.claude/skills/gstack` and runs `./setup` (compiles `browse`, registers all skills). Updates in place if already installed.
3. **Asks** before adding the recommended gstack section to CLAUDE.md.
4. **Asks** before enabling team mode (auto-update for a shared repo, which commits the change).
5. Verifies the build and tells you to restart.

It never edits CLAUDE.md or commits to your repo without explicit confirmation.

## Credit

gstack is built by [Garry Tan](https://github.com/garrytan) and licensed MIT. This installer is an independent wrapper, also MIT. All gstack functionality, updates, and support come from the upstream repo: https://github.com/garrytan/gstack

</details>
