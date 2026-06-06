---
name: anki-deck
description: Scaffold a new per-deck Anki skill under the shared cross-agent skills directory so the user can add cards to a custom deck from any agent. Use when the user wants to create a custom Anki deck workflow.
---

Scaffold a new per-deck Anki skill at `~/.agents/skills/anki-{slug}/SKILL.md`.

# Context

- Output: `~/.agents/skills/anki-{slug}/SKILL.md` (`{slug}` from § A1).
- Body source: `new-deck-template.md` (a ready-made, agent-neutral skill template in this repo), fetched in § A3 and filled in § A4.
- Rule: **Follow every step of the decision tree in order. Do not skip or reorder steps unless the user explicitly instructs otherwise.**

# Decision Tree

```
START
│
├─ [A0] Parse input — empty? ask → STOP ; else extract deck name/description → next
├─ [A1] Determine names — deck display name, slug, base tag, description → next
├─ [A2] ~/.agents/skills/anki-{slug}/SKILL.md exists? — YES → STOP ; NO → next
├─ [A3] Fetch template → next
├─ [A4] Fill placeholders → next
├─ [A5] Write file → next
└─ [A6] Report
```

# Actions

## A0. Parse Input

The user's input is a deck name, topic, or subject area (e.g. `한국사`, `일본어 N3`, `물리학`). If empty, ask what deck to create and **STOP**.

## A1. Determine Names

| Value | Rule | Example (`한국사`) |
|---|---|---|
| **Deck display name** | Clear name for the deck; use input as-is if good. | `한국사` |
| **Slug** | `anki-{slug}` folder and frontmatter `name`. Keep Korean as-is; English → lowercase-hyphenated. No spaces or path-invalid characters. | `한국사` |
| **Base tag** | Lowercase tag for all cards; hyphens for multi-word. | `한국사` |
| **Deck description** | Brief description, used in the skill's frontmatter. | `한국사 (Korean history)` |

## A2. Check Existing Skill

If `~/.agents/skills/anki-{slug}/SKILL.md` exists (Glob or Read), tell the user it already exists (delete the folder to recreate) and **STOP**. Otherwise proceed.

## A3. Fetch Template

```bash
curl -sL "https://raw.githubusercontent.com/jeheecheon/agent-skills/main/skills/anki-deck/new-deck-template.md"
```

Use the output as the new skill's body, including frontmatter (always the latest `main`).

## A4. Fill Placeholders

Replace these four tokens throughout the template:

| Placeholder | Replace with |
|---|---|
| `{{SLUG}}` | slug |
| `{{DECK_NAME}}` | deck display name |
| `{{DECK_DESCRIPTION}}` | deck description |
| `{{BASE_TAG}}` | base tag |

Replace **only** these four tokens. Leave Anki card-template syntax such as `{{Question}}`, `{{#Choices}}`, and `{{hint:Hint}}` untouched. The result is the complete, ready-to-write SKILL.md.

## A5. Write Skill

Write the built SKILL.md to `~/.agents/skills/anki-{slug}/SKILL.md` (`mkdir -p` first).

## A6. Report

> **Deck skill created!**
>
> - Deck: `{deck-display-name}` · Skill: `anki-{slug}`
> - File: `~/.agents/skills/anki-{slug}/SKILL.md`
>
> Run your skills-manager sync to make `anki-{slug}` available across your agents.

# Input

The user's request — a deck name, topic, or subject area. The agent derives naming from it.
