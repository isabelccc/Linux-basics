# 11. Permissions and Access Control

## Standard permissions

Three permission types: **Read (r)**, **Write (w)**, **Execute (x)**.

Three classes: **User (u)** owner, **Group (g)** owning group, **Others (o)** everyone else.

| Permission | On file | On directory |
|------------|--------|---------------|
| r | Read contents | List contents |
| w | Modify/delete | Add/remove/rename files |
| x | Run as program | Enter directory, access files inside |

Layout: `rwx rwx rwx` (user, group, others).

## Symbolic mode (chmod)

- **u, g, o, a** — user, group, others, all.
- **r, w, x** — read, write, execute.
- **+, -, =** — add, remove, set exactly.

Example: **`chmod u+x file`** adds execute for owner.

## Numeric (octal) mode

| Combination | Number |
|-------------|--------|
| --- | 0 |
| --x | 1 |
| -w- | 2 |
| -wx | 3 |
| r-- | 4 |
| r-x | 5 |
| rw- | 6 |
| rwx | 7 |

Three digits: user, group, others. Example: **`chmod 771 file`** = rwxrwx--x.

## Default permissions and umask

New file default often **666**, directory **777**. **umask** subtracts permissions (mask).

- **`umask`** — Show current umask (e.g. 0022).
- **`umask 0022`** — Set umask (e.g. files 644, dirs 755).
- **`umask u-x,g=r,o+w`** — Symbolic umask.

Common: 022 → files rw-r--r--, dirs rwxr-xr-x.

## Ownership

- **`chown user file`** — Change owner. **`chown user:group file`** — Owner and group.
- **`chgrp group file`** — Change group only.
