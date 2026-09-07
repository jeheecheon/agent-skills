---
name: wiki-research
description: Perform a web search to answer user queries, and optionally capture the raw extracted facts into the wiki's raw/ directory. Trigger when the user inputs "/wiki-research" or asks to search the web.
---

# 1. Terminology
- **Raw Repository**: Directory `raw/`.
- **Wiki**: Directory `wiki/`.

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Assert existence of Wiki, Raw Repository, and `AGENTS.md`
│  ├─ Missing → Instruct user to run `/wiki-topic-setup` → STOP
│  └─ Exists → [A1]
│
├─ [A1] Query Execution
│  │  Perform web search and output response
│  └─ OK → [A2]
│
├─ [A2] Capture Authorization
│  │  Prompt user for save authorization
│  ├─ User declines → STOP
│  └─ User affirms → [A3]
│
├─ [A3] Raw Artifact Generation
│  │  Generate unsummarized dump of facts in `raw/`
│  └─ OK → [A4]
│
└─ [A4] Ingestion Trigger
   │  Trigger `/wiki-ingest` on artifact
   └─ OK → END
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check for the existence of Wiki, Raw Repository, and `AGENTS.md`.
- Condition A (Missing): Output instruction to execute `/wiki-topic-setup` and halt.
- Condition B (Exists): Proceed to [A1].

## 3.2. [A1] Query Execution
- Action: Perform web search satisfying user query. Output comprehensive response applying the Citation Format. Proceed to [A2].

## 3.3. [A2] Capture Authorization
- Action: Prompt user: "Save these search results and sources as a new document in `raw/` and integrate into the wiki?".
  - Condition A (User affirms): Proceed to [A3].
  - Condition B (User declines): Halt execution.

## 3.4. [A3] Raw Artifact Generation
- Precondition: [A2] Condition A satisfied.
- Action: Generate `raw/search_YYYYMMDD_topic.md`.
- Invariant: Output must be an unsummarized dump of facts, quotes, and data to prevent data drift.
- Format Stricture:
  - Header: `# <Topic>`
  - Metadata: `- **Search Query:** <query>`, `- **Source URL:** <URL>`, `- **Fetched Date:** <YYYY-MM-DD>`
  - Body: `## Extracted Content` followed by raw data.
- Postcondition: Proceed to [A4].

## 3.5. [A4] Ingestion Trigger
- Action: Output confirmation of [A3] success. Proactively trigger `/wiki-ingest` on the generated artifact. Finish execution.

# 4. Formatting Constraints
Generated conversational responses in [A1] must strictly follow this exact citation template:

This sentence cites the first source[1](URL), and this cites the second source[2](URL). You must attach the anchor immediately after the word without any space, and reuse the same number and URL for the same source[1](URL).

---
- [1 - First source title](URL)
- [2 - Second source title](URL)
