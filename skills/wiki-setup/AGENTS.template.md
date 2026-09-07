# Wiki System Rules

## Architecture

- `raw/`: Immutable. Read-only.
- `wiki/`: Writeable. Maintain cross-references, resolve contradictions, ensure consistency.

## Index Contract

- Exhaustive: Unlisted content pages do not exist (excluding `log.md`).
- Missing entries: Ask the user. Do not search.

## Operations

Evaluate the user's intent and choose the appropriate action:

- **Simple Queries**: For general questions about the existing knowledge base, do NOT trigger any skill. Never read the whole wiki. Always read `wiki/index.md` first to find the relevant pages, load only those required, and answer directly.
- **`wiki-research` skill**: Trigger when the user asks a question that requires a web search. Perform the search, answer the user, and optionally capture the raw facts into the wiki.
- **`wiki-ingest` skill**: Trigger when the user provides new raw documents, articles, or text and wants to absorb/summarize them into the wiki.
- **`wiki-lint` skill**: Trigger when the user asks to check the wiki's health, validate links, fix formatting, or resolve contradictions.
- **`wiki-compound` skill**: Trigger when the user wants to preserve valuable chat synthesis, insights, architectural decisions, or conclusions reached during the conversation. Additionally, when you produce analytical conclusions, comparison tables, or novel cross-references during a response, proactively ask the user: "Save this to the wiki via `/wiki-compound`?".

## Output Contract

When answering queries based on sources, you MUST strictly format your response exactly like this template to ensure consistency:

This sentence cites the first source[1](URL), and this cites the second source[2](URL). You must attach the anchor immediately after the word without any space, and reuse the same number and URL for the same source[1](URL).

---
- [1 - First source title](URL)
- [2 - Second source title](URL)
