---
name: wiki-setup
description: Initialize a new LLM Wiki System in the current working directory. Trigger when the user inputs "/wiki-setup".
---

# 1. Terminology
- **Raw Repository**: Local directory `raw/` for immutable source documents. The LLM reads from here but never modifies these files.
- **Wiki**: Local directory `wiki/` containing markdown nodes.
- **AGENTS.md**: Agent instruction schema.

# 2. Decision Tree

```text
START
│
├─ [A0] Precondition Validation
│  │  Assert non-existence of `wiki/` and `wiki/index.md`
│  ├─ Existence detected → Prompt user for authorization to reinitialize → STOP
│  └─ Missing → [A1]
│
├─ [A1] Filesystem Initialization
│  │  Execute `mkdir -p raw wiki && echo "# Index" > wiki/index.md && echo "# Log" > wiki/log.md`
│  └─ OK → [A2]
│
└─ [A2] Schema Retrieval
   │  Check if `AGENTS.md` already contains `# Wiki System Rules`
   ├─ Already present → Skip retrieval → END
   ├─ Missing or absent → Execute `[ -f AGENTS.md ] && echo "" >> AGENTS.md; curl -sL "https://raw.githubusercontent.com/jeheecheon/agent-skills/main/skills/wiki-setup/AGENTS.template.md" >> AGENTS.md`
   │  ├─ OK → END
   │  └─ FAIL (non-zero exit) → Output network error → STOP
```

# 3. Actions

## 3.1. [A0] Precondition Validation
- Action: Check for the existence of `wiki/` and `wiki/index.md` in the current directory.
- Condition A (Existence detected): Prompt user for authorization to reinitialize. Halt execution.
- Condition B (Missing): Proceed to [A1].

## 3.2. [A1] Filesystem Initialization
- Action: Execute `mkdir -p raw wiki && echo "# Index" > wiki/index.md && echo "# Log" > wiki/log.md`.
- Postcondition: Baseline topology is instantiated. Proceed to [A2].

## 3.3. [A2] Schema Retrieval
- Action: Check if `AGENTS.md` already contains the header `# Wiki System Rules`.
  - Condition A (Already present): Schema already installed. Skip retrieval and finish execution.
  - Condition B (Missing or absent): Execute `[ -f AGENTS.md ] && echo "" >> AGENTS.md; curl -sL "https://raw.githubusercontent.com/jeheecheon/agent-skills/main/skills/wiki-setup/AGENTS.template.md" >> AGENTS.md`.
    - Success: Schema retrieved. Finish execution.
    - Failure: Network error or non-zero exit code. Output error and halt execution.
