# 1. Terminology

- **Workspace**: The root directory containing multiple topic subdirectories.
- **Topic Subdirectory**: An isolated wiki instance containing its own `raw/` and `wiki/` directories.
- **Subdirectory Index**: The registry of all active Topic Subdirectories, located at § 4.

# 2. Architecture Constraints

## 2.1. Execution Boundary
Creation of `raw/` or `wiki/` directories directly within the Workspace (§ 1) is strictly forbidden.

## 2.2. Context Awareness
Execution of any wiki skill (e.g., `ingest`, `lint`, `compound`, `research`) must be scoped to a specific Topic Subdirectory (§ 1). If the target is ambiguous, prompt the user for clarification before execution.

## 2.3. Index Maintenance
Whenever a new Topic Subdirectory is created, it must be appended to the Subdirectory Index (§ 4) in the format `- <directory_name>: <brief_description>`.

# 3. Operations Decision Tree

Start execution upon receiving user input.

```text
START
│
├─ [A0] Evaluate Intent
│  ├─ Request to create a new topic → [A1]
│  ├─ Request to execute a wiki skill → [A2]
│  ├─ Cross-topic query spanning multiple wikis → [A3]
│  └─ Unrelated query → Answer directly → END
│
├─ [A1] Topic Creation
│  │  Create the requested subdirectory within Workspace (§ 1)
│  ├─ Success → Navigate into subdirectory → Execute `/wiki-topic-setup` → Update Subdirectory Index (§ 2.3) → END
│  └─ Failure → Prompt user → STOP
│
├─ [A2] Skill Execution
│  │  Identify target Topic Subdirectory
│  ├─ Target identified → Execute requested skill relative to target subdirectory → END
│  └─ Target ambiguous → Prompt user for clarification → STOP
│
└─ [A3] Cross-Topic Aggregation
   │  Read `wiki/index.md` from all relevant Topic Subdirectories
   └─ Synthesize knowledge → Answer user → END
```

# 4. Subdirectory Index

<!-- Add new topics here in the format: - `directory_name`: description -->
