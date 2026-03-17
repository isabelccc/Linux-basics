# 12. Pipe and Redirect

## Standard streams

- **stdin** — Input (default: keyboard).
- **stdout** — Normal output (default: terminal).
- **stderr** — Error output (default: terminal).

## Redirects

- **`cmd > file`** — stdout to file (overwrite).
- **`cmd >> file`** — stdout to file (append).
- **`cmd 2> file`** — stderr to file.
- **`cmd < file`** — stdin from file.
- **`cmd 2>&1`** — stderr to same place as stdout.
- **`cmd &> file`** or **`cmd > file 2>&1`** — stdout and stderr to file.

## Pipe

**`|`** sends stdout of one command to stdin of the next.

```bash
w | grep user_name | sed s/user_name/admin/g
date | mail -s "Subject" user@host
```

**`|&`** — Pipe both stdout and stderr to the next command.

## Combining

- **`cmd > file 2>&1`** — Both streams to file.
- **`cmd | tee file`** — stdout to file and to screen.
- **`cmd 2>&1 | tee file`** — stdout and stderr to file and screen.
