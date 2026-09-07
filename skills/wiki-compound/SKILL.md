---
name: wiki-compound
description: Save valuable synthesis, decisions, or analysis from the chat history back into the wiki. Trigger when the user inputs "/wiki-compound".
---

# 1. Terminology
- **Synthesis**: Ephemeral analytical conclusions or decisions derived during chat.
- **Wiki**: Directory `wiki/`.
- **Index**: File `wiki/index.md`.
- **Log**: File `wiki/log.md`.

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Assert existence of Wiki and `AGENTS.md`
│  ├─ Missing → Instruct user to run `/wiki-setup` → STOP
│  └─ Exists → [A1]
│
├─ [A1] Value Extraction
│  │  Isolate Synthesis from conversation context
│  └─ OK → [A2]
│
├─ [A2] Content Drafting
│  │  Generate self-contained markdown section
│  └─ OK → [A3]
│
├─ [A3] Content Integration
│  │  Evaluate output destination
│  ├─ Relevant page exists → Inject content and cross-references → [A3.5]
│  ├─ Broad novel architecture → Generate dedicated page → [A3.5]
│  └─ Destination ambiguous → Prompt user → STOP
│
├─ [A3.5] Contradiction Detection
│  │  Compare injected content against existing wiki claims
│  ├─ No conflict → [A4]
│  ├─ Resolvable via timestamp → Annotate superseded claim inline → [A4]
│  └─ Ambiguous conflict → Flag with admonition and prompt user → [A4]
│
├─ [A4] Index Update
│  │  Append new page to Index under appropriate heading
│  └─ OK → [A5]
│
└─ [A5] Operation Logging
   │  Append entry to Log
   └─ OK → END
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check for the existence of Wiki and `AGENTS.md`.
- Condition A (Missing): Output instruction to execute `/wiki-setup` and halt.
- Condition B (Exists): Proceed to [A1].

## 3.2. [A1] Value Extraction
- Action: Isolate Synthesis from conversation context. Identify content matching any of these criteria:
  - **Analytical conclusions**: Novel insights derived from cross-referencing multiple wiki pages or sources.
  - **Architectural decisions**: Design choices, trade-offs evaluated, or rationale documented during discussion.
  - **Comparisons and evaluations**: Tables, pros/cons lists, or ranked assessments produced during Q&A.
  - **Discovered connections**: Previously unlinked relationships between entities or concepts.
  - **User-directed captures**: Content the user explicitly requests to preserve.
- Postcondition: Proceed to [A2].

## 3.3. [A2] Content Drafting
- Action: Generate a self-contained markdown section encapsulating the extracted value. Proceed to [A3].

## 3.4. [A3] Content Integration
- Action: Evaluate destination for [A2] output.
  - Condition A (Relevant page exists): Inject content and relative cross-references. Proceed to [A3.5].
  - Condition B (Broad novel architecture): Generate dedicated page. Proceed to [A3.5].
  - Failure Mode (Destination ambiguous): Halt and prompt user.

## 3.5. [A3.5] Contradiction Detection
- Action: For each page modified in [A3], compare injected content against pre-existing claims on that page.
  - Condition A (No conflict): Proceed to [A4].
  - Condition B (Resolvable): Newer synthesis supersedes older claim. Annotate the superseded claim inline (e.g. ~~old claim~~ with note citing the source of the new conclusion) and update with the current fact. Proceed to [A4].
  - Condition C (Ambiguous): Flag the contradiction with an admonition block (`> [!WARNING] Contradiction detected: ...`) and prompt user for resolution. Proceed to [A4] after resolution.

## 3.6. [A4] Index Update
- Action: If [A3] generated a page, append entry to Index under the appropriate heading.
- Invariant: Index exclusively targets leaf files. Proceed to [A5].

## 3.7. [A5] Operation Logging
- Action: Append `## [YYYY-MM-DD] compound | <Target>` to Log, enumerating modifications and any contradictions flagged. Finish execution.

# 4. Formatting Constraints
Generated content must satisfy:
1. **Deduplication**: One fact per document. Utilize relative markdown links.
2. **Numbering**: ATX headings require hierarchical decimal prefixes. Numbers are append-only.
3. **Partitioning**: Split files exceeding 300 lines.
4. **Index Topology**: Entries in Index follow `- [path/file.md](path/file.md) — <description>. Read when working on <task>.`
