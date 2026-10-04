# Claude Code Corporate Training

Bu repository, **Claude Code ile Kurumsal Yazılım Geliştirme Eğitimi** sırasında kullanılan demo prompt'larını ve uygulama akışlarını içerir.

Amaç; eğitim sırasında prompt'ları hızlıca paylaşmak, katılımcıların GitHub üzerinden kolayca kopyalamasını sağlamak ve eğitim sonrasında tekrar kullanılabilecek bir referans oluşturmaktır.

> Bu repository bir source-code projesi değildir.  
> Demo prompt'ları ve eğitim materyalleri için companion repository olarak kullanılır.

## Training Structure

```text
Day 1
Control the Agent
        ↓
Day 2
Control the Engineering Change
        ↓
Day 3
Control the Delivery System
```

Şu anda repository'de **Day 1** demo içerikleri bulunmaktadır.

## Day 1 — Control the Agent

| Module | Topic | Content |
|---|---|---|
| M01 | Agentic Working Model | [Open](day-1/M01.md) |
| M02 | Plan-first Development & Permissions | [Open](day-1/M02.md) |
| M03 | Context Engineering | [Open](day-1/M03.md) |
| M04 | Security, Privacy & Network Governance | [Open](day-1/M04.md) |
| M05 | Token, Context & Performance Management | [Open](day-1/M05.md) |

[Open Day 1 Guide](day-1/README.md)

## Day 1 Mental Model

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

## Demo Repository

Prompt'lar aşağıdaki training codebase üzerinde çalıştırılır:

```text
call-center-integration-platform
```

## Recommended Environment

```text
Claude Code
Sonnet 5
Java 21
Spring Boot
Maven Wrapper
Windows PowerShell
Git
```

Demo'ya göre Claude Code mode değişebilir:

```text
Plan Mode
Manual Mode
```

## How to Use This Repository

Her module aynı temel yapıyı izler:

```text
Learning Goal
→ Demo
→ Mode
→ Prompt
→ What to Observe
→ Discussion Point
→ Verification
```

Prompt'ları GitHub code block'larından kopyalayıp Claude Code terminaline yapıştırın.

> Output'un birebir aynı olması beklenmez. Model version, repository state ve mevcut context küçük farklılıklar üretebilir. Amaç aynı **engineering behavior**, **control model** ve **evidence discipline** üzerinde çalışmaktır.

## Core Principles

```text
Agentic ≠ Autonomous

Can do ≠ Allowed to do ≠ Approved to do

More Context ≠ Better Context

Relevant ≠ Shareable

Accessible ≠ Authorized

Capability ≠ Authorization
```

Performance mental model:

```text
Scope
→ Context
→ Tools
→ Model / Effort
→ Verification
```

## Security Note

Bu repository'deki security ve privacy örnekleri eğitim amaçlıdır.

- synthetic data kullanılır,
- real customer data kullanılmaz,
- real credentials veya secrets kullanılmaz,
- production system'lere bağlantı kurulmaz,
- network action gerektiğinde explicit human approval prensibi uygulanır.

## Repository Status

```text
Day 1 — Available
Day 2 — Coming later
Day 3 — Coming later
```
