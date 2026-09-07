# 1. Terminology

- **Raw Repository**: `raw/` directory. Immutable source documents.
- **Wiki**: `wiki/` directory. Editable Markdown nodes.
- **Index**: `wiki/index.md`. Exhaustive catalog of all wiki pages. Unlisted pages (excluding `wiki/log.md`) do not exist.
- **Log**: `wiki/log.md`. Chronological operation log.

# 2. Architecture Constraints

## 2.1. Access Controls
- Raw Repository (§ 1): Read-only. Modification is forbidden.
- Wiki (§ 1): Writeable. Must maintain cross-references and resolve contradictions.

## 2.2. Index Invariants
If a page is absent from Index (§ 1), ask the user. Do not execute unguided searches.

## 2.3. Log Read Optimization
When reading Log (§ 1) to understand recent operations, never load the entire file to prevent token exhaustion. Instead, use a terminal command (e.g., `tail -n 50 wiki/log.md` or `grep`) to read only the most recent entries.

# 3. Operations Decision Tree

Start execution upon receiving user input.

```text
START
│
├─ [A0] Evaluate Intent
│  ├─ Requires web search → Trigger `wiki-research` skill → END
│  ├─ Absorb new raw documents → Trigger `wiki-ingest` skill → END
│  ├─ Validate health, resolve contradictions → Trigger `wiki-lint` skill → END
│  ├─ Preserve chat synthesis or decisions → Trigger `wiki-compound` skill → END
│  └─ Query existing knowledge → [A1]
│
├─ [A1] Index Retrieval
│  │  Read Index (§ 1) to identify target pages. Do not read the entire Wiki.
│  ├─ Targets identified → Load target pages → [A2]
│  └─ Targets missing → Prompt user → STOP
│
├─ [A2] Query Resolution
│  │  Answer query adhering strictly to § 4.
│  └─ OK → [A3]
│
└─ [A3] Proactive Capture Evaluation
   │  Check if response contains novel synthesis or cross-references
   ├─ True → Prompt: "Save this to the wiki via `/wiki-compound`?" → END
   └─ False → END
```

# 4. Output Format

When citing sources during § 3 [A2], adhere to the following template.

## 4.1. Inline Citations
Attach the anchor immediately after the word without whitespace. Reuse the same number and URL for the identical source.
Example: `This sentence cites the first source[1](URL).`

## 4.2. Bibliography
Append a horizontal rule followed by a list of cited sources at the end of the response.
Example:
```markdown
---
- [1 - First source title](URL)
- [2 - Second source title](URL)
```
