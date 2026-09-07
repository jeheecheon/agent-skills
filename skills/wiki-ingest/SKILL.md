---
name: wiki-ingest
description: Process a raw source document and distribute its facts across the wiki. Trigger when the user inputs "/wiki-ingest".
---

# 1. Terminology
- **Raw Repository**: Directory `raw/`.
- **Wiki**: Directory `wiki/`.
- **Raw Document**: Immutable source material located in `raw/`.
- **Index**: The catalog file `wiki/index.md`.
- **Log**: The chronological record `wiki/log.md`.

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Assert existence of `raw/`, `wiki/`, and `AGENTS.md`
│  ├─ Missing → Instruct user to run `/wiki-topic-setup` → STOP
│  └─ Exists → [A1]
│
├─ [A1] Source Summarization
│  │  Generate summary file in `wiki/`
│  └─ OK → [A1.5]
│
├─ [A1.5] User Review
│  │  Present summary and planned fact distribution to user
│  ├─ User approves → [A2]
│  └─ User requests changes → Revise and re-present → [A1.5]
│
├─ [A2] Fact Distribution
│  │  Extract core topics from source
│  ├─ Relevant page exists → Inject facts and cross-references → [A3]
│  ├─ Topic warrants isolation → Generate dedicated page → [A3]
│  └─ Default → Retain facts exclusively in summary → [A3]
│
├─ [A3] Contradiction Detection
│  │  Compare newly injected facts against existing wiki claims
│  ├─ No conflict → [A4]
│  ├─ Resolvable via timestamp → Annotate superseded claim inline → [A4]
│  └─ Ambiguous conflict → Flag with admonition and prompt user → [A4]
│
├─ [A4] Index Update
│  │  Append new pages to Index under matching heading
│  └─ OK → [A5]
│
└─ [A5] Operation Logging
   │  Append entry to Log
   └─ OK → END
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check for the existence of `raw/`, `wiki/`, and `AGENTS.md`.
- Condition A (Missing): Output instruction to execute `/wiki-topic-setup` and halt.
- Condition B (Exists): Proceed to [A1].

## 3.2. [A1] Source Summarization
- Action: Parse target Raw Document. Generate a summary markdown file within `wiki/`. Proceed to [A1.5].

## 3.3. [A1.5] User Review
- Action: Present the generated summary and a brief outline of planned fact distribution targets (which existing pages will be updated, which new pages will be created) to the user.
  - Condition A (User approves): Proceed to [A2].
  - Condition B (User requests changes): Revise emphasis, scope, or targets per user feedback and re-present.

## 3.4. [A2] Fact Distribution
- Action: Extract core topics. For each topic:
  - Condition A (Relevant page exists in `wiki/`): Inject facts and relative cross-references.
  - Condition B (Topic warrants isolation): Generate a new dedicated page.
  - Default: Retain facts exclusively within the [A1] summary file.
- Postcondition: Proceed to [A3].

## 3.5. [A3] Contradiction Detection
- Action: For each page modified in [A2], compare newly injected facts against pre-existing claims on that page.
  - Condition A (No conflict): Proceed to [A4].
  - Condition B (Resolvable): Newer source supersedes older claim. Annotate the superseded claim inline (e.g. ~~old claim~~ with note citing the newer source) and update with the current fact. Proceed to [A4].
  - Condition C (Ambiguous): Flag the contradiction with an admonition block (`> [!WARNING] Contradiction detected: ...`) and prompt user for resolution. Proceed to [A4] after resolution.

## 3.6. [A4] Index Update
- Action: Append one list item per new page generated in [A1] or [A2] under the matching ATX heading in Index.
- Invariant: Index exclusively targets leaf files. Proceed to [A5].

## 3.7. [A5] Operation Logging
- Action: Append `- [YYYY-MM-DD] ingest | <Source Title> | <Modified Pages> | <Contradictions>` to Log as a single bullet item without line breaks. Finish execution.

# 4. Formatting Constraints
Generated content must satisfy:
1. **Deduplication**: One fact per document. Utilize relative markdown links.
2. **Numbering**: ATX headings (excluding Index and Log) require hierarchical decimal prefixes. Numbers are append-only.
3. **Partitioning**: Split files exceeding 300 lines.
4. **Index Topology**: Entries in Index follow `- [path/file.md](path/file.md) — <description>. Read when working on <task>.`
