# 31. Sed and Awk

## sed (stream editor)

**sed** applies editing commands to input line by line (file or stdin). Non-interactive.

- **Substitute:** **`sed 's/old/new/' file`** — Replace first occurrence per line. **`sed 's/old/new/g'`** — All occurrences (global).
- **`sed -i 's/old/new/g' file`** — Edit file in place (GNU sed).
- **Delete lines:** **`sed '/pattern/d' file`**.
- **Print specific lines:** **`sed -n '10p' file`** — Line 10. **`sed -n '1,5p' file`** — Lines 1–5.

## awk

**awk** processes text in lines and columns (fields). Default field separator is whitespace.

- **`awk '{print $1}' file`** — Print first field of each line.
- **`awk -F: '{print $1}' /etc/passwd`** — Use `:` as separator.
- **`awk '/pattern/ {print $0}' file`** — Print lines matching pattern.
- **`awk '$3 > 100' file`** — Lines where third field is greater than 100.

Both integrate well in pipelines (e.g. **grep | sed | awk**).
