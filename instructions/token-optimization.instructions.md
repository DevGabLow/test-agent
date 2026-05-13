---
description: "Token economy rules — reduce input/output token consumption across all agents via file-based communication and structured summaries. Applies to ALL agents and skills globally."
applyTo: ".github/agents/**/*.agent.md"
---

# Token Economy Protocol (RTK)

## Core: Files Over Prompts
NEVER pass full content between agents. ALWAYS use file system.

| ❌ Waste | ✅ Save |
|---|---|
| Pass full analysis in prompt | Pass file path; agent uses `read_file` |
| Return full code as response | Write to file; return summary only |
| Verbose confirmations | Single-line structured summary |

## Output Protocol
`{status} | {artifact} | {metrics}`

Status: `OK` | `OK_PARTIAL` | `FAIL` | `SKIP`
Metrics: `{N}f/{M}e/{A}auth`, `{N}H/{N}n/{B}B`, `{N} steps`

## Input Protocol
- Receive only identifiers (paths, tags), never full content
- Use `read_file` + line ranges to fetch context
- Use `grep_search` to locate symbols

## Token Budgets
| Context | Max | Strategy |
|---------|-----|----------|
| .agent.md system prompt | 500 | Terse bullets, no code templates |
| Subagent invocation | 200 | File paths + tag only |
| Agent response | 100 | Single summary line |
| File reads | Minimal | Line ranges, not full files |

## Compression
1. Bullets > prose
2. Tables > paragraphs
3. Paths > content
4. Codes > names (OK|FAIL|SKIP)
5. Numbers > descriptions (12H/3N/1B)
