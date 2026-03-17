# 2. Introduction to Shell

## What is a shell?

The Linux shell is a program that allows **text-based interaction** between the user and the operating system: you type commands and receive text responses. It is a powerful tool to navigate the system. When you log in, you start in your **home directory**.

- **Home directory** — Per-user; e.g. `/home/michael`. Represented by `~` (tilde). Other users cannot access your files here.
- **Command prompt** — Can show hostname, date, time, or current directory (`~` = home).

## Command and arguments

- **Command** — Runs a program to do a task (e.g. `echo` prints text).
- **Argument** — Input to the command (e.g. `echo hello`).
- **Options/flags** — Modify behavior (e.g. `echo -n hello` avoids a trailing newline).

Example: `uptime` needs no argument; it prints how long the system has been running.

## Command types

1. **Internal (built-in)** — Part of the shell (e.g. `echo`, `cd`, `pwd`, `set`). About 30 such commands.
2. **External** — Separate programs, often under `/usr/bin` or installed via packages (e.g. `mv`, `date`, `cp`).

To see if a command is internal or external:

```bash
type echo
type date
```

## Shell as interpreter

The shell sits between you and the OS:

- **User input** — Commands and data from the keyboard.
- **Shell** — Interprets commands and talks to the OS.
- **OS** — Runs programs and manages hardware.

## Common shells

| Shell | Description |
|-------|-------------|
| **bash** | Default on most Linux distros; scripting and compatibility. |
| **zsh** | Rich features, autocompletion, theming. |
| **ksh** | Korn shell; scripting. |
| **sh** | Original Bourne shell; portable, minimal. |

List installed shells: `cat /etc/shells`.  
Current shell: `echo $SHELL`.
