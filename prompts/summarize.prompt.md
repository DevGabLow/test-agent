---
description: "Compress verbose content into a token-efficient structured summary. Use when: reducing agent output before passing to another agent, summarizing large documents, creating context digests."
argument-hint: "Content type and target token budget"
---

Summarize the provided content into a compact, structured format. Follow the Token Economy Protocol.

## Compression Rules
1. **Tables over prose** — convert paragraphs to markdown tables where possible
2. **Bullets over sentences** — use `-` lists, not full paragraphs
3. **Paths over content** — reference file locations instead of pasting
4. **Codes over descriptions** — use short codes: `OK|FAIL|SKIP`, `H|N|B|A` (happy/negative/boundary/auth)
5. **Numbers over words** — `12H/3N/1B` instead of verbose descriptions

## Output Format
Return ONLY the compressed summary, no preamble or explanation:

```
{short_status} | {key_counts} | {critical_issues_if_any}
```

## Content to Summarize
{{content}}
