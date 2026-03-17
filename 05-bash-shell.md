# 5. Bash Shell

## Shell types

Common shells: Bourne (sh), C (csh/tsh), Korn (ksh), Z (zsh), **Bash** (Bourne Again).

- Check current shell: **`echo $SHELL`**
- Change default shell: **`chsh`** (then log in again).

## Bash features

- **Tab** — Auto-complete commands and paths; press twice to list options.
- **Aliases** — `alias dt=date` then run `dt`.
- **History** — `history` lists previous commands.

## Environment variables

- List all: **`env`**
- Set for current session: **`VAR=value`**
- Export to child processes: **`export VAR=value`**
- Persist: add **`export VAR=value`** to **`~/.profile`** or **`~/.bashrc`** (e.g. `echo "export OFFICE=caleston" >> ~/.profile`).
- View: **`echo $VAR`** (e.g. `echo $SHELL`, `echo $LOGNAME`).

## PATH

When you run an external command, the shell looks for it in the directories listed in **`PATH`** (colon-separated). To add a directory:

```bash
export PATH=$PATH:/your/dir
```

For a permanent change, add that line to `~/.profile` or `~/.bashrc`.

## Configuration files

- **`/etc/shells`** — List of valid login shells.
- **`~/.bashrc`** — Sourced for interactive non-login shells (aliases, prompts).
- **`~/.profile`** or **`~/.bash_profile`** — Sourced at login (exports, PATH).

## Identifying your shell

- **`echo $SHELL`** — Login shell.
- **`ps -p $$`** — Process for current shell.
