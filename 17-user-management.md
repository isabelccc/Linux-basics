# 17. User Management

## User info files

- **`/etc/passwd`** — Username, UID, GID, home directory, default shell. Password field is usually `x` (real hash in `/etc/shadow`).
- **`/etc/shadow`** — Encrypted passwords, expiration; readable only by root.

View users: **`less /etc/passwd`**. List usernames: **`awk -F: '{print $1}' /etc/passwd`**.

## Root and sudo

- **root** — Superuser for system administration. Avoid logging in as root; use **sudo** instead.
- Add user to sudo: **`usermod -aG sudo user`** (Debian/Ubuntu) or **`usermod -aG wheel user`** (RHEL/CentOS).
- Edit sudoers safely: **`visudo`**. Example: **`%wheel ALL=(ALL) ALL`** or **`user ALL=NOPASSWD:/sbin/reboot`**.

## Switching users

- **`su`** — Switch to root (or **`su -`** for login shell). **`su - user`** — Switch to another user.
- **`sudo cmd`** — Run command as root (if allowed by sudoers).

## User and group commands

- **`useradd`** / **`adduser`** — Create user. **`userdel`** — Remove user.
- **`usermod`** — Modify user (e.g. shell, groups). **`chsh`** — Change login shell.
- **`groupadd`**, **`groupdel`**, **`groupmod`** — Manage groups.
- **`id`** — Show UID, GID, groups. **`groups`** — List user’s groups.
