# 27. Package Managers

## Concepts

Packages are archives containing software and metadata. Package managers install, update, and remove them and resolve dependencies.

- **RPM-based** — RHEL, CentOS, Fedora: **rpm** (low-level), **yum** or **dnf** (high-level).
- **Deb-based** — Debian, Ubuntu: **dpkg** (low-level), **apt** or **apt-get** (high-level).

## RPM / yum / dnf

- **`rpm -i package.rpm`** — Install. **`rpm -q name`** — Query. **`rpm -e name`** — Erase.
- **`yum install name`** or **`dnf install name`** — Install. **`yum remove name`** — Remove.
- **`yum search term`** — Search. **`yum list installed`** — List installed.

## dpkg / apt

- **`dpkg -i package.deb`** — Install. **`dpkg -l`** — List. **`dpkg -r name`** — Remove.
- **`apt update`** — Refresh package lists. **`apt install name`** — Install. **`apt remove name`** — Remove.
- **`apt search term`** — Search. **`apt list --installed`** — List installed.

**apt** is the recommended user-facing tool on Debian/Ubuntu; **apt-get** is script-friendly.
