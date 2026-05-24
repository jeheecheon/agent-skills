---
name: claude-aliases
description: Configure shell aliases for quick Claude model access in zsh or bash. Use when the user wants terminal shortcuts for Claude models.
---

Configure shell aliases for quick Claude model access (`haiku`, `sonnet`, `opus`) in the user's shell configuration file. This command installs three convenience functions that let the user invoke Claude models directly from the terminal.

# Context

## Purpose

After running this command, the user will be able to type `haiku`, `sonnet`, or `opus` in their terminal to interact with Claude:

- **With arguments** (single-shot): `sonnet "translate hello to French"` → prints Claude's answer and returns.
- **Without arguments** (interactive): `opus` → prompts the user to type a question, then prints Claude's answer.

The functions call the `claude` CLI binary with the appropriate `--model` flag and `--print` (non-interactive) flag, piping the user's input into Claude and printing the result.

## What the aliases look like after installation

**For zsh** (the user's zsh config file will receive this block):

```zsh
# claude-aliases
unalias haiku sonnet opus 2>/dev/null
haiku()  { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Haiku  --print "$input"; else claude --model Haiku  --print "$*"; fi; }
sonnet() { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Sonnet --print "$input"; else claude --model Sonnet --print "$*"; fi; }
opus()   { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Opus   --print "$input"; else claude --model Opus   --print "$*"; fi; }
alias haiku='noglob haiku'
alias sonnet='noglob sonnet'
alias opus='noglob opus'
# /claude-aliases
```

**For bash** (bash lacks `vared` and `noglob`; use bash-compatible substitutions):

```bash
# claude-aliases
unalias haiku sonnet opus 2>/dev/null
haiku()  { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Haiku  --print "$input"; else claude --model Haiku  --print "$*"; fi; }
sonnet() { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Sonnet --print "$input"; else claude --model Sonnet --print "$*"; fi; }
opus()   { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Opus   --print "$input"; else claude --model Opus   --print "$*"; fi; }
# /claude-aliases
```

## Glossary (terms used in this document)

| Term | Meaning |
|---|---|
| `$SHELL` | Environment variable containing the **path** to the user's login shell (e.g., `/bin/zsh`). This is set at login and does NOT change when the user runs a different shell interactively. |
| Login shell | The shell started at login. On macOS Terminal.app, every new window is a login shell. On Linux, the first tty is a login shell, subsequent terminals are non-login. |
| Non-login interactive shell | A shell started after login (e.g., when opening a new GNOME Terminal tab on Linux). |
| `~/.zshrc` | Zsh reads this on **every** interactive shell start (both login and non-login). |
| `~/.bash_profile` | Bash reads this on **login** shell start only. macOS Terminal.app defaults to this. |
| `~/.bashrc` | Bash reads this on **non-login interactive** shell start only. Most Linux distros default to this. |
| `vared` | Zsh built-in: **v**ariable **ed**itor. Prompts for input with an editable default. Syntax: `vared -p "prompt: " varname`. Not available in bash. |
| `noglob` | Zsh built-in prefix that disables filename globbing for the next command. Used as `alias foo='noglob foo'` so that `foo *.txt` passes the literal string `*.txt` to the function instead of expanding it to a file list. Not available in bash. |
| `read -p` | Bash built-in: reads a line from stdin with a prompt. Syntax: `read -p "prompt: " varname`. |
| `echo -n` | Suppresses the trailing newline that `echo` normally adds. |
| `echo -e` | Enables interpretation of backslash escapes (`\n`, `\t`, etc.) in the string. Bash-specific flag. |
| Heredoc | Shell construct `cat >> file << 'DELIM' ... DELIM` that writes a multi-line block. Single-quoted delimiter (`'DELIM'`) prevents variable expansion inside the block. |
| Idempotency marker | A comment like `# claude-aliases` that lets this command detect its own previous installation and skip re-appending. |
| Bash tool subshell | Claude Code's Bash tool runs commands in a fresh child shell each invocation. Changes to environment variables, aliases, or functions inside the Bash tool do NOT affect the user's interactive terminal. This is a hard OS-level constraint (a child process cannot modify its parent's environment). |

## Rules

- **Follow every step of the decision tree in order. Do not skip or reorder steps unless the user explicitly instructs otherwise.**
- Always back up the config file before modifying it (action [A11]).
- If any step after [A11] fails, rollback from backup (actions [A14] and [A15]).
- Use a single-quoted heredoc delimiter (`'ALIASES'`) when appending; this prevents `$1`, `$*`, `$#`, etc. inside the function bodies from being expanded at install time.
- Never tell the user that aliases are "active now" after running this command. Sourcing happens in a subshell and cannot affect the user's terminal. Always instruct the user to run `source {config_file}` manually or open a new terminal.

# Decision Tree

```
START
│
├─ [A0] Parse input
│  │  $ARGUMENTS is ignored (this command takes no arguments)
│  └─ next
│
├─ [A1] Detect operating system
│  │  Run: uname -s
│  │  Store result as OS_NAME (values: "Darwin" for macOS, "Linux" for Linux, other)
│  └─ next
│
├─ [A2] Detect shell
│  │  Step 1: Read $SHELL environment variable
│  │  Step 2: If empty, fall back to passwd lookup
│  │  Step 3: Extract shell name from path (basename)
│  │
│  ├─ "zsh"                          → SHELL_NAME="zsh"  → next
│  ├─ "bash"                         → SHELL_NAME="bash" → next
│  ├─ empty / undetectable           → [A3] → STOP
│  └─ "fish"/"ksh"/"dash"/"csh"/etc. → [A3] → STOP
│
├─ [A4] Determine config file path
│  ├─ SHELL_NAME = "zsh":
│  │  └─ CONFIG_FILE = ~/.zshrc → next
│  └─ SHELL_NAME = "bash":
│     ├─ OS_NAME = "Darwin":
│     │  ├─ ~/.bash_profile exists → CONFIG_FILE = ~/.bash_profile → next
│     │  └─ otherwise              → CONFIG_FILE = ~/.bashrc       → next
│     └─ OS_NAME ≠ "Darwin":
│        └─ CONFIG_FILE = ~/.bashrc → next
│
├─ [A5] Ensure config file exists
│  │  Run: test -f {CONFIG_FILE} && echo EXISTS || echo MISSING
│  ├─ EXISTS  → next
│  └─ MISSING → create empty file via `touch {CONFIG_FILE}`
│     ├─ OK   → next
│     └─ FAIL → [A6] → STOP
│
├─ [A7] Verify config file is writable
│  │  Run: test -w {CONFIG_FILE} && echo WRITABLE || echo READONLY
│  ├─ WRITABLE → next
│  └─ READONLY → [A6] → STOP
│
├─ [A8] Classify installation status
│  │  marker_count  = grep -c "^# claude-aliases$" {CONFIG_FILE}
│  │  func_count    = grep -cE "^(haiku|sonnet|opus)[[:space:]]*\(\)" {CONFIG_FILE}
│  │
│  │  | marker_count | func_count | status     |
│  │  |--------------|------------|------------|
│  │  | ≥ 1          | any        | INSTALLED  |
│  │  | 0            | ≥ 1        | CONFLICT   |
│  │  | 0            | 0          | CLEAN      |
│  │
│  ├─ INSTALLED → [A9]  → STOP
│  ├─ CONFLICT  → [A10] → STOP
│  └─ CLEAN     → next
│
├─ [A11] Back up config file
│  │  BACKUP_PATH = {CONFIG_FILE}.bak.{timestamp}
│  │  Run: cp {CONFIG_FILE} {BACKUP_PATH}
│  └─ next
│
├─ [A12] Normalize trailing newline
│  │  Ensure the file ends with a newline so the appended block starts
│  │  on a fresh line. Otherwise the marker could be glued to the last line.
│  └─ next
│
├─ [A13] Append shell-appropriate alias block
│  ├─ SHELL_NAME = "zsh"  → append zsh heredoc block
│  └─ SHELL_NAME = "bash" → append bash heredoc block
│
├─ [A14] Verify append succeeded
│  │  Re-run: grep -c "^# claude-aliases$" {CONFIG_FILE}
│  ├─ ≥ 1 → next
│  └─ 0   → rollback from BACKUP_PATH → [A15] → STOP
│
├─ [A16] Syntax-check the modified config in a fresh subshell
│  │  zsh:  zsh  -c 'source ~/.zshrc  && echo SYNTAX_OK'
│  │  bash: bash -c 'source ~/.bashrc && echo SYNTAX_OK'
│  │  (NOTE: This runs in a Bash-tool subshell. It does NOT activate
│  │   aliases in the user's interactive terminal.)
│  ├─ Output contains "SYNTAX_OK" → next
│  └─ Syntax error                → rollback from BACKUP_PATH → [A15] → STOP
│
├─ [A17] Instruct user to activate aliases
│  │  The user MUST manually source the config file in their terminal,
│  │  OR open a new terminal. Aliases cannot be activated from inside
│  │  the Bash tool (child process cannot modify parent shell).
│  └─ next
│
└─ [A18] Report summary
```

# Actions

## A0. Parse Input

This command takes no arguments. `$ARGUMENTS` is the slash-command argument placeholder that Claude Code substitutes when the command is invoked; for this command it may be empty or populated, but must be ignored either way.

Proceed to A1.

## A1. Detect Operating System

Run via Bash tool:

```bash
uname -s
```

- Output `Darwin` → `OS_NAME="Darwin"` (macOS)
- Output `Linux` → `OS_NAME="Linux"`
- Other output (e.g., `FreeBSD`, `OpenBSD`, `CYGWIN_NT-10.0`, etc.) → treat as `OS_NAME="Other"`.

Store `OS_NAME` for use in A4. Proceed to A2.

## A2. Detect Shell

### Step 1 — read `$SHELL`

```bash
echo "$SHELL"
```

`$SHELL` contains the path to the user's login shell as recorded in `/etc/passwd` (or the Directory Services database on macOS). It is typically set for every user.

### Step 2 — fallback if empty

If the output of Step 1 is empty or unset, fall back to the operating system user database:

- **macOS** (`OS_NAME="Darwin"`):
  ```bash
  dscl . -read /Users/$(whoami) UserShell 2>/dev/null | awk '{print $2}'
  ```
- **Linux / other** (`OS_NAME="Linux"` or other):
  ```bash
  getent passwd "$(whoami)" | cut -d: -f7
  ```
- **Last-resort fallback** (works on both but may report the current process shell, not the login shell):
  ```bash
  ps -p $$ -o comm= 2>/dev/null
  ```

Use whichever produces a non-empty result.

### Step 3 — extract shell name

Parse the shell name from the full path by taking the basename:

```bash
basename "$SHELL_PATH"
```

Examples:
- `/bin/zsh` → `zsh`
- `/opt/homebrew/bin/bash` → `bash`
- `/usr/local/bin/fish` → `fish`

### Step 4 — classify

- Shell name contains `zsh` → `SHELL_NAME="zsh"`, proceed to A4.
- Shell name contains `bash` → `SHELL_NAME="bash"`, proceed to A4.
- Shell name is empty or undetectable → proceed to A3.
- Shell name is anything else (`fish`, `ksh`, `dash`, `csh`, `tcsh`, `sh`, etc.) → proceed to A3.

## A3. Unsupported or Undetectable Shell

Tell the user:

> **This command only supports `zsh` and `bash`.**
>
> Detected shell: `{shell_name_or_empty}`
>
> To install the Claude aliases manually, add the appropriate block to your shell's configuration file. See the `# Context → What the aliases look like after installation` section of this command's source at `plugins/misc/commands/claude-aliases.md` for the full snippet.
>
> If you believe your shell is supported and detection failed, run this command again after exporting `SHELL`:
> ```bash
> export SHELL=/path/to/your/shell
> ```

**STOP — cannot proceed.**

## A4. Determine Config File Path

### Why the config file depends on shell AND operating system

Different shells read different startup files, and the same shell reads different files depending on whether it's started as a login shell or non-login interactive shell. This matters because:

- On **macOS**, Terminal.app (and iTerm2 by default) starts every new window as a **login** shell. Bash on a login shell reads `~/.bash_profile` first; it only reads `~/.bashrc` if `~/.bash_profile` explicitly sources it. So on macOS, most users put their aliases in `~/.bash_profile`.
- On **Linux**, most desktop terminals (GNOME Terminal, Konsole, xterm, etc.) start bash as a **non-login interactive** shell, which reads `~/.bashrc` and does NOT read `~/.bash_profile`.
- **Zsh** is simpler: it always reads `~/.zshrc` for any interactive shell, login or not. So `~/.zshrc` is the correct file for zsh regardless of OS.

### Decision

| `SHELL_NAME` | `OS_NAME` | `CONFIG_FILE` |
|---|---|---|
| `zsh` | any | `~/.zshrc` |
| `bash` | `Darwin` and `~/.bash_profile` exists | `~/.bash_profile` |
| `bash` | `Darwin` and `~/.bash_profile` does NOT exist | `~/.bashrc` |
| `bash` | `Linux` / other | `~/.bashrc` |

Run via Bash tool to probe `~/.bash_profile` existence:

```bash
test -f ~/.bash_profile && echo EXISTS || echo MISSING
```

Store the resolved path as `CONFIG_FILE`. Proceed to A5.

## A5. Ensure Config File Exists

Check whether the resolved config file exists:

```bash
test -f "{CONFIG_FILE}" && echo EXISTS || echo MISSING
```

- `EXISTS` → proceed to A7.
- `MISSING` → create an empty file:
  ```bash
  touch "{CONFIG_FILE}"
  ```
  - If `touch` succeeds (exit code 0), proceed to A7.
  - If `touch` fails (e.g., parent directory does not exist, permission denied, filesystem read-only), proceed to A6.

## A6. File Access Error

Tell the user, substituting `{CONFIG_FILE}` and `{reason}`:

> **Cannot write to `{CONFIG_FILE}`.**
>
> Reason: {reason — e.g., "file is read-only", "permission denied", "parent directory does not exist"}
>
> Please check:
> 1. The file's permissions: `ls -l {CONFIG_FILE}`
> 2. The parent directory exists and is writable: `ls -ld $(dirname {CONFIG_FILE})`
> 3. You own the file: `stat {CONFIG_FILE}`
>
> Fix the permissions issue and retry this command.

**STOP — cannot proceed.**

## A7. Verify Config File Is Writable

```bash
test -w "{CONFIG_FILE}" && echo WRITABLE || echo READONLY
```

- `WRITABLE` → proceed to A8.
- `READONLY` → proceed to A6 with reason "file is not writable (permission denied)".

## A8. Classify Installation Status

Determine whether the aliases are already installed, conflicting, or clean.

### Count the marker comment

```bash
grep -c "^# claude-aliases$" "{CONFIG_FILE}"
```

- The marker `# claude-aliases` (exactly, at the start of a line) is written by this command during A13.
- If count ≥ 1, the block has been installed by this command previously.

### Count the function definitions (in case of manual/legacy installs)

```bash
grep -cE "^(haiku|sonnet|opus)[[:space:]]*\(\)" "{CONFIG_FILE}"
```

- This detects lines like `haiku()`, `sonnet ()`, `opus   ()` at the start of a line.
- If this count is ≥ 1 but the marker count is 0, the user has manually defined these functions without using this command. Installing over them would create duplicate definitions.

### Classify

| marker_count | func_count | status | action |
|---|---|---|---|
| ≥ 1 | any | `INSTALLED` | proceed to A9 |
| 0 | ≥ 1 | `CONFLICT` | proceed to A10 |
| 0 | 0 | `CLEAN` | proceed to A11 |

## A9. Already Installed

Tell the user:

> **Claude aliases are already installed in `{CONFIG_FILE}`.**
>
> The marker `# claude-aliases` was found in your config file.
>
> To reinstall:
> 1. Open `{CONFIG_FILE}` in your editor.
> 2. Delete the block between `# claude-aliases` and `# /claude-aliases`.
> 3. Save the file.
> 4. Run this command again.
>
> To use the already-installed aliases immediately, run:
> ```
> source {CONFIG_FILE}
> ```

**STOP — already installed.**

## A10. Conflict (Manual Definitions Exist)

Tell the user:

> **Conflict detected: your `{CONFIG_FILE}` already defines `haiku`, `sonnet`, or `opus` functions without the `# claude-aliases` marker.**
>
> Installing on top of existing definitions would create duplicate function bodies. Please resolve the conflict by choosing one of:
>
> 1. **Keep your existing definitions** — do nothing. Your manual aliases will continue to work.
> 2. **Replace with the managed block** — manually delete the existing `haiku()`, `sonnet()`, `opus()` function definitions (and any related `alias` or `unalias` lines) from `{CONFIG_FILE}`, then run this command again.
>
> This command will not auto-remove your existing definitions because it cannot distinguish your customizations from the default shape.

**STOP — conflict must be resolved manually.**

## A11. Back Up Config File

Create a timestamped backup so the modification can be reverted if any subsequent step fails.

```bash
BACKUP_PATH="{CONFIG_FILE}.bak.$(date +%Y%m%d%H%M%S)"
cp "{CONFIG_FILE}" "$BACKUP_PATH"
echo "$BACKUP_PATH"
```

- Store the backup path (e.g., `~/.zshrc.bak.20260410142530`) for use in A14 and A16 rollback.
- If `cp` fails, treat this as a filesystem error and proceed to A6 with reason "backup failed".

Proceed to A12.

## A12. Normalize Trailing Newline

If the config file does not end with a newline character, appending a heredoc directly would glue the first line of the new block onto the final pre-existing line. Check and fix:

```bash
if [ -s "{CONFIG_FILE}" ] && [ "$(tail -c 1 "{CONFIG_FILE}")" != "" ]; then
  printf '\n' >> "{CONFIG_FILE}"
fi
```

Explanation:
- `[ -s file ]` is true if the file is non-empty.
- `tail -c 1` reads the last byte.
- If the last byte is not an empty string (i.e., the shell substitution trimmed a newline — shells trim trailing newlines in command substitution), append one newline.

Proceed to A13.

## A13. Append Alias Block

Append the shell-appropriate block using a heredoc with a **single-quoted delimiter** so that variable expansion is suppressed inside the block. This is critical: without single quotes, `$#`, `$*`, `$input`, etc. inside the function bodies would be expanded at install time.

### For zsh (`SHELL_NAME="zsh"`)

```bash
cat >> "{CONFIG_FILE}" << 'ALIASES'

# claude-aliases
unalias haiku sonnet opus 2>/dev/null
haiku()  { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Haiku  --print "$input"; else claude --model Haiku  --print "$*"; fi; }
sonnet() { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Sonnet --print "$input"; else claude --model Sonnet --print "$*"; fi; }
opus()   { if [[ $# -eq 0 ]]; then local input=""; vared -p "prompt: " input; echo -n "Responding...👻\n"; claude --model Opus   --print "$input"; else claude --model Opus   --print "$*"; fi; }
alias haiku='noglob haiku'
alias sonnet='noglob sonnet'
alias opus='noglob opus'
# /claude-aliases
ALIASES
```

**Why each piece exists:**

- **Leading blank line** — separates the block visually from preceding config.
- **`# claude-aliases`** — idempotency marker. [A8] greps for this to detect prior installs.
- **`unalias haiku sonnet opus 2>/dev/null`** — removes any pre-existing aliases with these names before defining functions. `2>/dev/null` suppresses errors if the aliases don't exist yet.
- **Function bodies** — each of the three functions checks `$# -eq 0` (argument count is zero):
  - **Zero args** → declare `local input=""`, read user input via `vared -p "prompt: " input` (zsh's editable prompt), print `Responding...👻` followed by a newline, then call `claude --model {Model} --print "$input"`.
  - **One or more args** → call `claude --model {Model} --print "$*"` directly, where `$*` is the concatenation of all arguments.
  - `--print` tells the `claude` CLI to run non-interactively and output the result, then exit.
- **About `echo -n "Responding...👻\n"` in zsh** — in zsh, the `echo` built-in interprets `\n` as a newline character by default (unlike bash's `echo`, which requires `-e` for this). So `echo -n "...\n"` prints the text followed by exactly one newline: the `\n` in the string becomes the newline, and `-n` suppresses the extra trailing newline that `echo` would otherwise add on top. Net result: one newline total.
- **`alias name='noglob name'`** — prevents argument globbing. Without this, typing `haiku list all *.txt files` would expand `*.txt` to a list of filenames in the current directory before the function ever sees it. With `noglob`, the literal string `*.txt` is preserved.
- **`# /claude-aliases`** — closing marker. Lets the user (or a future uninstall script) locate the end of the block.

### For bash (`SHELL_NAME="bash"`)

Bash does not have `vared` or `noglob`. Substitute:
- `vared -p "prompt: " input` → `read -p "prompt: " input` (bash built-in).
- `echo -n "Responding...👻\n"` → `echo "Responding...👻"` (bash's echo adds a trailing newline by default; the `\n` in zsh's version was to produce the newline, which bash's unadorned `echo` does automatically).
- Drop the `noglob` aliases — bash has no `noglob` built-in. The trade-off: bash users will experience glob expansion on arguments containing `*`, `?`, or `[...]`. If the user types `haiku what is *`, bash will expand `*` before calling the function. This is a known limitation; no clean bash equivalent exists without wrapping every invocation in `set -f` / `set +f`.

```bash
cat >> "{CONFIG_FILE}" << 'ALIASES'

# claude-aliases
unalias haiku sonnet opus 2>/dev/null
haiku()  { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Haiku  --print "$input"; else claude --model Haiku  --print "$*"; fi; }
sonnet() { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Sonnet --print "$input"; else claude --model Sonnet --print "$*"; fi; }
opus()   { if [[ $# -eq 0 ]]; then local input=""; read -p "prompt: " input; echo "Responding...👻"; claude --model Opus   --print "$input"; else claude --model Opus   --print "$*"; fi; }
# /claude-aliases
ALIASES
```

Proceed to A14.

## A14. Verify Append Succeeded

Re-check the marker to confirm the write actually persisted:

```bash
grep -c "^# claude-aliases$" "{CONFIG_FILE}"
```

- Output ≥ 1 → proceed to A16.
- Output = 0 → the write did not take effect. Rollback:
  ```bash
  cp "$BACKUP_PATH" "{CONFIG_FILE}"
  ```
  Then proceed to A15.

## A15. Rollback Error Report

Tell the user (substitute context):

> **Installation failed and was rolled back.**
>
> Step: {A14 or A16}
> Reason: {append did not persist / syntax error after append}
> Backup restored from: `{BACKUP_PATH}`
>
> Please check:
> - The file is writable and the filesystem is not read-only.
> - No other process is holding the file open.
> - For syntax errors: the original file has been restored, so inspect the contents manually and compare against the alias block shown in this command's Context section.

**STOP — installation rolled back.**

## A16. Syntax-Check the Modified Config

Spawn a fresh subshell that sources the config file end-to-end. If there is a syntax error anywhere in the file (not just in the appended block — a pre-existing error would also show up here), this will catch it.

Use the resolved `{CONFIG_FILE}` from A4 in the command — do NOT hardcode `~/.zshrc` or `~/.bashrc`, because on macOS bash the file might be `~/.bash_profile`.

- **For zsh:**
  ```bash
  zsh -c 'source "{CONFIG_FILE}" 2>&1 && echo SYNTAX_OK' 2>&1
  ```
- **For bash:**
  ```bash
  bash -c 'source "{CONFIG_FILE}" 2>&1 && echo SYNTAX_OK' 2>&1
  ```

### Important clarification about sourcing

The Bash tool runs each command in a **fresh child shell**. Sourcing a file inside that child shell:
- Loads the file into the child's memory, allowing syntax errors to be detected.
- **Does NOT** affect the user's interactive terminal, because a child process cannot modify its parent's environment. This is an OS-level constraint, not a shell quirk.

Therefore, the purpose of this step is **only syntax validation**, not activation. Activation happens in A17 via instructions to the user.

### Interpretation

- Output contains `SYNTAX_OK` → proceed to A17.
- Output contains error messages (e.g., `parse error`, `syntax error near unexpected token`, `command not found: vared`) → rollback from `$BACKUP_PATH` and proceed to A15. Note that `vared: command not found` on bash would indicate this command incorrectly wrote the zsh block to a bash config, which is a bug in this command — do not just warn the user; treat it as a fatal error and rollback.

## A17. Instruct User to Activate Aliases

Present this to the user exactly (substitute `{CONFIG_FILE}`):

> **Installed. To activate in your current terminal, run:**
>
> ```
> source {CONFIG_FILE}
> ```
>
> Or simply open a new terminal window — the aliases will load automatically.
>
> **Why can't this command activate them for you?**
> The `claude` CLI runs shell commands in a child process. A child process cannot modify its parent shell's environment, aliases, or functions — this is an OS-level constraint. So the file has been written and its syntax validated, but the aliases only take effect once your interactive shell re-reads the file.

Proceed to A18.

## A18. Report Summary

Present the final status report to the user:

```
┌──────────────────────────────────────────────┐
│        Claude Aliases — Installation         │
├──────────────────────────────────────────────┤
│  OS              {OS_NAME}                   │
│  Shell           {SHELL_NAME}                │
│  Config file     {CONFIG_FILE}               │
│  Backup          {BACKUP_PATH}               │
│  Functions       haiku, sonnet, opus         │
│  Marker          # claude-aliases            │
│  Syntax check    PASSED                      │
│  Activation      Manual (see above)          │
└──────────────────────────────────────────────┘
```

Then show usage examples:

> **Usage after activation:**
>
> ```
> haiku "what is 2+2?"              # single-shot with Haiku
> sonnet "translate hi to French"   # single-shot with Sonnet
> opus                              # interactive prompt mode with Opus
> opus "explain monads briefly"     # single-shot with Opus
> ```
>
> **To uninstall later:**
> Delete the block between `# claude-aliases` and `# /claude-aliases` in `{CONFIG_FILE}`, then re-source the file or open a new terminal.

# Input

`$ARGUMENTS` — ignored. This command takes no arguments. Any argument passed to the slash command is silently discarded.
