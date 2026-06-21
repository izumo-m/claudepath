# claudepath

[![CI](https://github.com/Mahiler1909/claudepath/actions/workflows/ci.yml/badge.svg)](https://github.com/Mahiler1909/claudepath/actions/workflows/ci.yml)
[![Coverage](https://codecov.io/gh/Mahiler1909/claudepath/branch/main/graph/badge.svg)](https://codecov.io/gh/Mahiler1909/claudepath)
[![PyPI version](https://img.shields.io/pypi/v/claudepath)](https://pypi.org/project/claudepath/)
[![Downloads](https://img.shields.io/pypi/dm/claudepath)](https://pypi.org/project/claudepath/)
[![Python versions](https://img.shields.io/pypi/pyversions/claudepath)](https://pypi.org/project/claudepath/)
[![License](https://img.shields.io/github/license/Mahiler1909/claudepath)](LICENSE)

Move or rename [Claude Code](https://claude.ai/claude-code) projects without losing session history, memory, and context.

> [!NOTE]
> **Unofficial patched fork** of [`Mahiler1909/claudepath`](https://github.com/Mahiler1909/claudepath), maintained for personal use by [@izumo-m](https://github.com/izumo-m). Not affiliated with or endorsed by the upstream author. The badges and the PyPI/Homebrew install commands below refer to the **upstream** package.
>
> **What this fork changes**
> - Integrates upstream open PRs ahead of merge: [#3](https://github.com/Mahiler1909/claudepath/pull/3) (preserve file mtime) and [#4](https://github.com/Mahiler1909/claudepath/pull/4) (encoder fix for [#2](https://github.com/Mahiler1909/claudepath/issues/2)).
> - Disables the PyPI self-update check and `claudepath update` (running it would overwrite this fork with the upstream build).
> - Version labelled `1.1.1+izumo` so it is distinguishable from stock releases.
>
> **Install this fork from source** (not from PyPI):
> ```bash
> git clone https://github.com/izumo-m/claudepath.git
> cd claudepath
> pipx install .   # or: pip install .
> ```

## The Problem

When you move or rename a project directory, Claude Code loses all your session history because sessions are keyed to the absolute path of your project. You end up with orphaned data in `~/.claude/projects/` and a fresh start.

**claudepath** fixes this by updating all path references in Claude Code's data files after a move.

## Installation

```bash
# With pipx (recommended — isolated install)
pipx install claudepath

# With pip
pip install claudepath
```

### Homebrew (macOS/Linux)

```bash
brew tap Mahiler1909/tools
brew install claudepath
```

## Usage

### Move a project

Moves the directory on disk **and** updates all Claude Code references:

```bash
claudepath mv ~/old/location/my-project ~/new/location/my-project
```

### Remap after a manual move

If you already moved the directory yourself, just update Claude's references:

```bash
claudepath remap ~/old/location/my-project ~/new/location/my-project
```

### Preview changes (dry run)

See exactly what would change without modifying anything:

```bash
claudepath mv ~/old/path ~/new/path --dry-run
```

### List tracked projects

```bash
claudepath list
```

### Update claudepath

Self-update is **disabled** in this fork — running it from PyPI would overwrite the fork with the upstream build. Update from git instead:

```bash
git switch main && git pull
```

Running `claudepath update` just prints this reminder.

### Restore from backup

List and restore from automatic backups:

```bash
claudepath restore --list       # see available backups
claudepath restore              # restore the latest backup
claudepath restore 20260227_145300  # restore a specific backup
```

### Merge sessions when destination already has Claude data

If you opened Claude Code at the new location before running `claudepath remap`, Claude Code will have already created a new project directory there. Use `--merge` to combine the sessions from both directories:

```bash
claudepath remap ~/old/path ~/new/path --merge
```

Without `--merge`, claudepath will fail with a clear error suggesting you add the flag.

### Full help

```bash
claudepath help
claudepath mv --help
```

## What it updates

| File / Directory | What changes |
|---|---|
| `~/.claude/projects/{encoded-dir}/` | Renamed to match new path |
| `sessions-index.json` | `originalPath`, `projectPath`, `fullPath` updated (proper JSON parsing) |
| `{session}.jsonl` files | `cwd` and file path references updated (line-by-line, handles large files) |
| Subagent `.jsonl` files | Same as above, recursive |
| `~/.claude/history.jsonl` | `project` field updated for all matching entries |
| `usage-data/session-meta/*.json` | `project_path` updated (token usage & analytics) |

> **Note:** `file-history/`, `todos/`, `tasks/`, and `shell-snapshots/` are keyed by session UUID, not by project path — they don't need updating.

## Options

| Flag | Description |
|---|---|
| `--dry-run` | Preview all changes without writing anything |
| `--no-backup` | Skip creating a backup before modifying files |
| `--yes` / `-y` | Skip the confirmation prompt |
| `--merge` | Merge sessions when destination already has Claude data |
| `--verbose` / `-v` | Show detailed file-by-file processing output |
| `--claude-dir <path>` | Override the Claude data directory (default: `~/.claude`) |

## Environment Variables

| Variable | Description |
|---|---|
| `NO_COLOR` | Disable colored output (set to any value) |
| `FORCE_COLOR` | Force colored output even when not a TTY |

## Backup & Rollback

By default, claudepath creates a backup before making any changes:

```
~/.claude/backups/claudepath/{timestamp}/
```

The backup includes:
- The full project data directory (`~/.claude/projects/{encoded}/`)
- `~/.claude/history.jsonl`
- Affected `usage-data/session-meta/*.json` files (token usage & analytics)
- When using `--merge`: both the source and destination project directories

If any step fails, claudepath automatically restores from the backup. Use `--no-backup` only if you already have your own backup or want to skip the extra time.

## How Claude Code encodes paths

Claude Code stores project data in `~/.claude/projects/` using an encoded directory name: every `/` in the absolute path is replaced with `-`.

```
/Users/foo/my-project  →  -Users-foo-my-project
```

This means moving `/Users/foo/old-name` to `/Users/foo/new-name` requires:
1. Renaming `-Users-foo-old-name` to `-Users-foo-new-name`
2. Updating all path strings inside the data files

## License

MIT
