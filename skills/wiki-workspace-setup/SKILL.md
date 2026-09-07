---
name: wiki-workspace-setup
description: Initialize a multi-topic LLM Wiki root workspace. Trigger when the user inputs "/wiki-workspace-setup".
---

# 1. Terminology
- **Workspace**: The root directory containing multiple independent topic wikis.
- **AGENTS.md**: Agent instruction schema for the workspace.

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Check if `AGENTS.md` already contains `# Multi-Topic Wiki Workspace`
│  ├─ Already present → Skip retrieval → END
│  └─ Missing or absent → [A1]
│
└─ [A1] Schema Retrieval
   │  Execute `[ -f AGENTS.md ] && echo "" >> AGENTS.md; curl -sL "https://raw.githubusercontent.com/jeheecheon/agent-skills/main/skills/wiki-workspace-setup/AGENTS.template.md" >> AGENTS.md`
   ├─ OK → END
   └─ FAIL (non-zero exit) → Output network error → STOP
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check if `AGENTS.md` in the current directory already contains the header `# Multi-Topic Wiki Workspace`.
- Condition A (Already present): Schema already installed. Skip retrieval and finish execution.
- Condition B (Missing or absent): Proceed to [A1].

## 3.2. [A1] Schema Retrieval
- Action: Execute `[ -f AGENTS.md ] && echo "" >> AGENTS.md; curl -sL "https://raw.githubusercontent.com/jeheecheon/agent-skills/main/skills/wiki-workspace-setup/AGENTS.template.md" >> AGENTS.md`.
  - Success: Schema retrieved. Finish execution.
  - Failure: Network error or non-zero exit code. Output error and halt execution.
