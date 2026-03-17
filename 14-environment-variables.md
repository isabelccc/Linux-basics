# 14. Environment Variables

## Types

- **Shell variables** — Current session only (e.g. `count=5` in scripts).
- **Environment variables** — Exported with **`export VAR=value`**; child processes inherit them (e.g. PATH, HOME, LANG).

## Viewing and setting

- **`printenv`** or **`env`** — List environment variables.
- **`echo $VAR`** — Print value of VAR.
- **`export VAR=value`** — Set and export. Add to **`~/.profile`** or **`~/.bashrc`** to persist.

## Common variables

- **PATH** — Colon-separated list of directories searched for executables.
- **HOME** — User's home directory (~).
- **PWD** — Current working directory.
- **USER** / **LOGNAME** — Current username.
- **SHELL** — Login shell.
- **HISTSIZE** — Number of commands kept in history (0 disables).
- **LANG** — Locale (e.g. en_US.UTF-8).

## PATH

When you run a command, the shell looks in each directory in **PATH**. To add a directory:

```bash
export PATH=$PATH:/your/directory
```
