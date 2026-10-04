# Day 1 — Control the Agent

Day 1'in amacı, Claude Code ile çalışırken agent davranışını, context'i, execution boundary'lerini ve resource kullanımını kontrollü biçimde yönetmektir.

## Modules

| ID | Module | Main Question |
|---|---|---|
| M01 | [Agentic Working Model](M01.md) | Agent repository'de ne yapıyor? |
| M02 | [Plan-first Development & Permissions](M02.md) | Agent ne zaman hareket edebilir? |
| M03 | [Context Engineering](M03.md) | Kararı hangi bilgi şekillendiriyor? |
| M04 | [Security, Privacy & Network Governance](M04.md) | Agent neye gerçekten yetkili? |
| M05 | [Token, Context & Performance Management](M05.md) | Aynı işi nasıl daha verimli yapar? |

## Day 1 Flow

```text
M01
Understand the Agent
        ↓
M02
Control the Agent
        ↓
M03
Control its Context
        ↓
M04
Control its Boundaries
        ↓
M05
Control its Resource Usage
```

## Recommended Baseline

```text
Model       → Sonnet 5
Auto-memory → OFF
Repository  → clean working tree
Planning    → Plan Mode
Execution   → Manual Mode
```

Before each module, verify repository state when appropriate:

```powershell
git status --short
```

Expected:

```text
<empty>
```
