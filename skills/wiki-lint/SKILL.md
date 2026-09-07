---
name: wiki-lint
description: Validate wiki integrity and resolve inconsistencies. Trigger when the user inputs "/wiki-lint".
---

# 1. Terminology
- **Wiki**: Directory `wiki/`.
- **Index**: File `wiki/index.md`.
- **Log**: File `wiki/log.md`.
- **Orphan**: A markdown file lacking inbound links from other content pages (excluding Index).

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Assert existence of Wiki and `AGENTS.md`
│  ├─ Missing → Instruct user to run `/wiki-topic-setup` → STOP
│  └─ Exists → [A1]
│
├─ [A1] Index Validation
│  │  Enforce bidirectional consistency between Wiki and Index
│  └─ OK → [A2]
│
├─ [A2] Topology Health
│  │  Detect broken links, Orphans, and missing cross-references
│  └─ OK → [A3]
│
├─ [A3] Fact Resolution
│  │  Detect contradictions across Wiki
│  ├─ Resolvable via timestamp priority → Update with newest fact → [A4]
│  └─ Ambiguous resolution → Prompt user → STOP
│
├─ [A4] Format Validation
│  │  Assert decimal prefixes and file length limits
│  └─ OK → [A5]
│
├─ [A5] Coverage Analysis
│  │  Detect implicit concepts, stale claims, and knowledge gaps
│  └─ OK → [A6]
│
└─ [A6] Operation Logging
   │  Append entry to Log
   └─ OK → END
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check for the existence of Wiki and `AGENTS.md`.
- Condition A (Missing): Output instruction to execute `/wiki-topic-setup` and halt.
- Condition B (Exists): Proceed to [A1].

## 3.2. [A1] Index Validation
- Action: Enforce bidirectional consistency between Wiki contents and Index entries.
  - Inject missing Wiki pages into Index.
  - Purge Index entries pointing to nonexistent files.
- Postcondition: Proceed to [A2].

## 3.3. [A2] Topology Health
- Action: Detect broken links, Orphans, and missing cross-references.
  - Remove or repair unresolved links.
  - Inject inbound links from relevant parents to Orphans.
  - Identify pages that discuss related topics but lack mutual cross-references. Suggest or inject the missing links.
- Postcondition: Proceed to [A3].

## 3.4. [A3] Fact Resolution
- Action: Detect contradictions across Wiki.
  - Condition A (Safe): Contradiction resolvable via timestamp priority. Update with newest fact. Preserve deprecated fact as historical trace. Proceed to [A4].
  - Condition B (Unsafe): Resolution ambiguous. Halt and prompt user.

## 3.5. [A4] Format Validation
- Action: Assert all headings in Wiki (excluding Index and Log) utilize hierarchical decimal prefixes. Flag files exceeding 300 lines.
- Postcondition: Proceed to [A5].

## 3.6. [A5] Coverage Analysis
- Action: Scan all Wiki pages and perform three checks:
  - **Implicit Concepts**: Identify terms or entities referenced across multiple pages (≥3 occurrences) that lack a dedicated page. Suggest page creation for each.
  - **Stale Claims**: Detect claims whose source material has been superseded by newer ingested sources. Flag with `> [!NOTE] Potentially stale — see [newer-source.md](newer-source.md)`.
  - **Knowledge Gaps**: Identify topics with thin coverage or unanswered questions. For each gap, suggest a specific search query and prompt the user: "Run `/wiki-research` to fill this gap?".
- Output: Present findings as a summary list to the user. Proceed to [A6].

## 3.7. [A6] Operation Logging
- Action: Append `- [YYYY-MM-DD] lint | <Summary>` to Log as a single bullet item without line breaks, including counts of issues found/resolved. Finish execution.
