# 🐚 Linux Shell Scripting & Automation

## 📋 Table of Contents
- [1️⃣ The Shebang (#!)](#1️⃣-the-shebang-)
- [2️⃣ Safety First: Bash Strict Mode](#2️⃣-safety-first-bash-strict-mode)
- [3️⃣ Variables & Parameter Expansion](#3️⃣-variables--parameter-expansion)
- [4️⃣ Control Flow & Logic](#4️⃣-control-flow--logic)
- [5️⃣ Input Parsing (getopts)](#5️⃣-input-parsing-getopts)
- [6️⃣ Advanced: Trap & Signal Handling](#6️⃣-advanced-trap--signal-handling)
- [7️⃣ Arrays & Maps](#7️⃣-arrays--maps)
- [8️⃣ Debugging Workflow](#8️⃣-debugging-workflow)

---

## 1️⃣ The Shebang (#!)

The first line tells the kernel **which interpreter** to use.

| Shebang | Meaning |
|:--------|:--------|
| `#!/bin/bash` | Hardcoded path. Fails if bash is in `/usr/local/bin` (e.g., FreeBSD). |
| `#!/usr/bin/env bash` | **Recommended**. Searches `$PATH` for the first `bash`. |
| `#!/bin/sh` | POSIX shell. Restricted features (no arrays, no `[[ ]]`). |

---

## 2️⃣ Safety First: Bash Strict Mode

By default, Bash is dangerous. It continues even if a command fails.
Always start your scripts with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

**Breakdown:**
- **`set -e`**: Exit immediately if a command exits with a non-zero status.
- **`set -u`**: Treat unset variables as an error. (Main preventer of `rm -rf / $UNSET_VAR`).
- **`set -o pipefail`**: If `grep` fails in `cat file | grep text | wc -l`, the whole pipeline fails.

---

## 3️⃣ Variables & Parameter Expansion

Basics are easy (`NAME="Paul"`).
**Advanced Expansion** is where the power lies.

| Syntax | Meaning | Example (`FILE="image.jpg"`) |
|:-------|:--------|:-----------------------------|
| `${VAR:-default}` | Use default if unset. | `NAME=${INPUT:-"Guest"}` |
| `${VAR#pattern}` | Remove **Prefix**. | `${FILE#*.} -> jpg` (Extension) |
| `${VAR%pattern}` | Remove **Suffix**. | `${FILE%.*} -> image` (Basename) |
| `${VAR//from/to}` | Find & Replace. | `${VAR// /_}` (Replace spaces with _) |

---

## 4️⃣ Control Flow & Logic

### The "New" Test (`[[ ]]`)
Always use `[[ ]]` instead of `[ ]`. It handles spaces and logic safer.

```bash
if [[ -f "$FILE" && "$USER" == "root" ]]; then
    echo "DANGER: You are root."
fi
```

### Loops
```bash
# C-Style Loop
for ((i=0; i<10; i++)); do
    echo "Count: $i"
done

# Iterating Files (Safe)
for file in *.log; do
    [[ -e "$file" ]] || break  # Handle case where no files exist
    echo "Processing $file..."
done
```

---

## 5️⃣ Input Parsing (getopts)

Professional scripts take flags (`-f`, `-v`).

```bash
while getopts "hu:v" opt; do
  case "$opt" in
    h) echo "Usage: script.sh -u <user>"; exit 0 ;;
    u) USERNAME=$OPTARG ;;
    v) VERBOSE=true ;;
    *) echo "Invalid option"; exit 1 ;;
  esac
done
```

---

## 6️⃣ Advanced: Trap & Signal Handling

What if the user hits `CTRL+C` while your script is writing a temp file?
**Trap** catches the signal and runs a function.

```bash
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/temp_*
}

# Trap EXIT (Runs on success OR failure OR CTRL+C)
trap cleanup EXIT

touch /tmp/temp_data
echo "Working..."
sleep 10
```

---

## 7️⃣ Arrays & Maps

**Indexed Arrays (Ordered lists):**
```bash
fruits=("Apple" "Banana" "Cherry")
echo "${fruits[1]}"  # Banana
echo "${fruits[@]}"  # All elements
```

**Associative Arrays (Key-Value Maps) (Bash 4+):**
```bash
declare -A user_info
user_info[name]="Alice"
user_info[role]="Admin"

echo "${user_info[role]}"
```

---

## 8️⃣ Debugging Workflow

1.  **Syntax Check**: `bash -n script.sh` (Dry run).
2.  **Trace Mode**: `bash -x script.sh` (Prints every command before running).
3.  **ShellCheck**: Use [shellcheck.net](https://www.shellcheck.net/) or vscode extension. It spots 99% of bugs.

---

> 🧭 **Next:** [Advanced Topics: Cron & Environment Variables →](./10-advanced-topics.md)