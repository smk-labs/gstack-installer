---
name: install-gstack
description: Use when the user runs `/gstack-installer:install-gstack`, or asks to "install gstack", "set up gstack", "add Garry Tan's Claude Code setup", "get the gstack skills", or "update gstack". Bootstraps gstack (github.com/garrytan/gstack) by running its real upstream installer: clones the repo to ~/.claude/skills/gstack and runs `./setup`, which compiles the `browse` browser binary and registers all 70+ skills natively. gstack is NOT a normal plugin (its skills live at the repo root and hardcode ~/.claude/skills/gstack/bin paths, and it needs a Bun build step), so it cannot be loaded by `/plugin install` directly. This skill is the bridge. It is idempotent: if gstack is already present it updates in place instead of reinstalling. After running, it optionally adds the gstack section to CLAUDE.md and offers team mode (auto-update for a shared repo). Does not edit CLAUDE.md or commit to a repo without explicit confirmation.
argument-hint: "[--team]"
allowed-tools: Bash, AskUserQuestion
---

# Install or update gstack

This skill installs [gstack](https://github.com/garrytan/gstack) the way its author intends: by running the upstream `./setup` build, which is the only thing that compiles the `browse` browser engine and wires up gstack's ~60 `bin/` tools. This plugin does not vendor or fork gstack; it triggers gstack's own installer so you always get the current upstream version.

Work through the steps in order. Report what you run. Stop only at the two confirmation gates (CLAUDE.md edit, team-mode commit) and at any hard blocker.

## Step 0: Preflight

Detect the platform and required tools. Run:

```bash
echo "OS: $(uname -s)"
echo "git:  $(command -v git  || echo MISSING)"
echo "bun:  $(command -v bun  || echo MISSING)"
echo "node: $(command -v node || echo MISSING)"
```

- `git` and `bun` are required on every platform. `node` is additionally required on Windows.
- If `bun` is MISSING, stop and show the user gstack's checksum-verified install snippet, then exit:

  ```
  BUN_VERSION="1.3.10"
  tmpfile=$(mktemp)
  curl -fsSL "https://bun.sh/install" -o "$tmpfile"
  echo "Verify checksum before running: shasum -a 256 $tmpfile"
  BUN_VERSION="$BUN_VERSION" bash "$tmpfile" && rm "$tmpfile"
  ```

- If `git` is MISSING (or `node` is missing on Windows), tell the user to install it and stop.

## Step 1: Detect existing install

```bash
if [ -d ~/.claude/skills/gstack ]; then echo "PRESENT"; else echo "ABSENT"; fi
```

- **ABSENT** -> fresh install (Step 2a).
- **PRESENT** -> gstack is already installed. Do NOT clobber it. Update in place (Step 2b). Mention that the canonical update path, once gstack is loaded, is the `/gstack-upgrade` skill.

## Step 2a: Fresh install

Run the upstream installer verbatim:

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup
```

`./setup` builds the browser binary and registers skills with Claude Code. It can print a lot of output; that is normal.

## Step 2b: Update existing install

```bash
cd ~/.claude/skills/gstack && git pull --ff-only && ./setup
```

If `git pull` reports local changes or a non-fast-forward, do not force anything. Report it and let the user decide (they may have customized gstack).

## Step 3: CLAUDE.md section (ask first)

gstack works best when CLAUDE.md tells Claude to route browsing through it. This writes to a file, so confirm with AskUserQuestion before writing. Offer: global `~/.claude/CLAUDE.md`, the project `./CLAUDE.md`, or skip.

Then run the block below, setting `TARGET` to the chosen path. It is idempotent (skips if a gstack section is already present) and creates the file if it does not exist. Keep the section text verbatim.

```bash
TARGET="$HOME/.claude/CLAUDE.md"   # or "./CLAUDE.md" for the project
if grep -q '/office-hours' "$TARGET" 2>/dev/null; then
  echo "gstack section already present in $TARGET, skipping"
else
  mkdir -p "$(dirname "$TARGET")"
  cat >> "$TARGET" <<'EOF'

## gstack

Use the /browse skill from gstack for all web browsing. Never use mcp__claude-in-chrome__* tools. Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /connect-chrome, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /setup-gbrain, /retro, /investigate, /document-release, /document-generate, /codex, /cso, /autoplan, /plan-devex-review, /devex-review, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, /learn.
EOF
  echo "appended gstack section to $TARGET"
fi
```

## Step 4: Team mode (ask first, only if `--team` or user opts in)

Team mode makes a shared repo auto-install gstack for every teammate and commits that change. It runs `git commit`, so confirm with AskUserQuestion first and only proceed on an explicit yes. Requirements: the current directory is inside a git repo, and gstack is installed (Steps 2a/2b done).

Default (block teammates without gstack) runs verbatim:

```bash
(cd ~/.claude/skills/gstack && ./setup --team) && ~/.claude/skills/gstack/bin/gstack-team-init required && git add .claude/ CLAUDE.md && git commit -m "require gstack for AI-assisted work"
```

If the user prefers to nudge rather than block, swap `required` for `optional` in that command. Do not push; leave pushing to the user.

## Step 5: Verify and finish

```bash
test -x ~/.claude/skills/gstack/browse/dist/browse && echo "browse binary: OK" || echo "browse binary: NOT BUILT (re-run ./setup; check Bun)"
ls ~/.claude/skills/gstack | head
cat ~/.claude/skills/gstack/VERSION 2>/dev/null
```

Then tell the user: **restart Claude Code (or reload the window)** so the ~70 gstack skills load. Suggest a first command: `/office-hours`, `/review`, or `/qa <url>`.

## Notes

- **Update later:** run `/gstack-upgrade` (ships with gstack) or re-run this skill.
- **Uninstall:** `~/.claude/skills/gstack/bin/gstack-uninstall` (falls back to `rm -rf ~/.claude/skills/gstack`).
- **What runs on your machine:** this skill executes gstack's upstream `./setup`, which builds with Bun and writes under `~/.claude/skills/gstack` and `~/.gstack`. Anyone who wants to audit it first can read the repo at github.com/garrytan/gstack before approving the clone.
- **Other agents/hosts:** gstack also targets Codex, OpenCode, Cursor, Factory, etc. via `./setup --host <name>`. This skill installs the Claude Code host by default.
