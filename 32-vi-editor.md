# 32. Vi Editor

**vi** (and **vim**, its improved version) is a console text editor available on almost all Linux systems. Many systems make **vi** a link to **vim**.

Open a file: **`vi filename`** or **`vim filename`**.

## Modes

1. **Command mode** — Default when you open a file. Keystrokes are commands (move, delete, copy, etc.), not text.
2. **Insert mode** — Type text. Enter with **i**, **I**, **a**, **A**, **o**, **O**. Leave with **Esc**.
3. **Last line mode** — Enter with **:**. Used to save, quit, search, set options. Leave with **Esc**.

## Basic commands (command mode)

- **i** — Insert before cursor. **a** — Insert after. **o** — New line below. **O** — New line above.
- **x** — Delete character. **dd** — Delete line. **yy** — Yank (copy) line. **p** — Paste.
- **w** — Forward by word. **b** — Back by word. **0** — Start of line. **$** — End of line. **G** — End of file.
- **/pattern** — Search forward. **?pattern** — Search backward. **n** — Next match.

## Last line mode (:)

- **:w** — Save. **:q** — Quit. **:wq** or **:x** — Save and quit. **:q!** — Quit without saving.
- **:set number** — Show line numbers. **:n** — Go to line n.
