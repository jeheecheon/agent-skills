---
name: anki-deck
description: Create a new Anki deck command file so the user can add cards to a custom deck via a dedicated Anki command. Use when the user wants to scaffold a custom deck workflow.
---

Create a new Anki deck command file so the user can add cards to a custom deck via `/anki:{deck-name}`.

# Context

- Purpose: Generate a new command file under `~/.claude/commands/` (user-scope) by fetching `new-deck-template.md` from GitHub
- File naming: `~/.claude/commands/anki:{command-name}.md` — the `anki:` prefix in the filename makes the command callable as `/anki:{command-name}` in Claude Code.
- After this command runs, the user can invoke `/anki:{command-name}` to add cards to the new deck.
- Rule: **Follow every step of the decision tree in order. Do not skip or reorder steps unless the user explicitly instructs otherwise.**

# Decision Tree

```
START
│
├─ [A0] Parse input
│  ├─ Input empty? → ask user for deck name/description → STOP
│  └─ Input provided → extract deck name and description → next
│
├─ [A1] Determine names
│  │  Derive: deck display name, command file name, base tag
│  └─ next
│
├─ [A2] Command file already exists?
│  │  Check: ~/.claude/commands/anki:{command-name}.md
│  ├─ YES → tell user the command already exists → STOP
│  └─ NO → next
│
├─ [A3] Fetch template
│  │  Fetch: https://raw.githubusercontent.com/jeheecheon/claude-marketplace/main/plugins/anki/new-deck-template.md
│  └─ next
│
├─ [A4] Fill template
│  │  Replace all placeholders with derived values
│  └─ next
│
├─ [A5] Write command file
│  │  Write to: ~/.claude/commands/anki:{command-name}.md
│  └─ next
│
└─ [A6] Report
```

# Actions

## A0. Parse Input

`$ARGUMENTS` contains the user's input — a deck name, topic description, or subject area.

Examples:
- `한국사` → deck about Korean history
- `일본어 N3` → deck about Japanese JLPT N3
- `물리학` → deck about physics

If input is empty, ask the user what deck they want to create and **STOP**.

## A1. Determine Names

From the parsed input, derive three values:

| Value | Rule | Example (input: `한국사`) |
|---|---|---|
| **Deck display name** | A clear, concise name for the Anki deck. Use the input as-is if it's already a good name; otherwise craft a descriptive name. | `한국사` |
| **Command file name** | The deck name portion used in the filename `anki:{name}.md`. Use the input directly if it's simple; for multi-word inputs use hyphens. The full path becomes `~/.claude/commands/anki:{name}.md`. | `한국사` |
| **Base tag** | Lowercase tag for all cards in this deck. Use hyphens for multi-word. | `한국사` |
| **Deck description** | A brief description of what this deck covers, used in the command file's Context section. | `한국사 (Korean history)` |

**Naming guidelines:**
- The command name should match what the user typed as closely as possible so `/anki:{name}` feels intuitive.
- For Korean input, keep the Korean name as-is (e.g., `한국사` not `korean-history`).
- For English input, use lowercase with hyphens (e.g., `linear-algebra`).

## A2. Check Existing Command

Check if the file `~/.claude/commands/anki:{command-name}.md` already exists using the Glob or Read tool.

- If it exists (Glob returns a match, or Read returns file content): tell the user **"The command `/anki:{command-name}` already exists. Use it directly or delete the file to recreate."** → **STOP**
- If it does not exist (Glob returns empty list, or Read returns an error): proceed.

## A3. Fetch Template

Fetch the template from GitHub via Bash:

```bash
curl -sL "https://raw.githubusercontent.com/jeheecheon/claude-marketplace/main/plugins/anki/new-deck-template.md"
```

Use the output as the template content. This URL always points to the latest version on the `main` branch.

## A4. Fill Template

Replace all placeholders in the template with the derived values:

| Placeholder | Replace with |
|---|---|
| `{{DECK_NAME}}` | Deck display name |
| `{{DECK_DESCRIPTION}}` | Deck description |
| `{{BASE_TAG}}` | Base tag |

## A5. Write Command File

Write the filled template to:

```
~/.claude/commands/anki:{command-name}.md
```

The `anki:` prefix in the filename is what makes it callable as `/anki:{command-name}` in Claude Code.

## A6. Report

Tell the user:

> **Deck command created!**
>
> - Deck name: `{deck-display-name}`
> - Command: `/anki:{command-name}`
> - File: `~/.claude/commands/anki:{command-name}.md`
>
> You can now use `/anki:{command-name} {topic}` to add cards to the **{deck-display-name}** deck.
>
> **Note:** The command is immediately available since it's in user-scope (`~/.claude/commands/`). No reload needed.

# Input

`$ARGUMENTS` — a deck name, topic, or subject area description. The agent derives appropriate naming from this input.
