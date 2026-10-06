# Final Workshop — Complete Operating Model'i Çalıştır

## Senaryo

**Priority Customer Escalation Workflow**

```text
Incoming Call
→ IVR Integration
→ CRM Customer Context
→ Bot / AI Decision Boundary
→ Escalation Decision
→ Notification Integration
```

## Amaç

Üç gün boyunca öğrenilen operating model'i tek bir team-scale engineering workflow içinde uygulamak.

## Constraints

```text
Production access yok
Real customer data yok
Real credentials yok
Deployment yok
Force push yok
Automatic merge yok
Major boundary'lerde Human approval
```

## Önerilen Team Structure

### Squad A

```text
IVR
+ CRM
```

### Squad B

```text
Bot / AI decision boundary
+ Notification
```

Shared contract:

```text
Customer / Call Context
```

## Önerilen Roller

- Workstream Owner
- Implementation Driver
- Test & Verification Owner
- Delivery Evidence Owner

---

# Workshop Operating Model

```text
Requirement
→ Architecture Analysis
→ Team Decomposition
→ Task Ownership
→ Contract Freeze
→ Plan
→ Human Approval
→ Isolated Implementation
→ Testing
→ Debugging / Correction
→ Automated Controls
→ Git Diff
→ Atomic Commit
→ Pull Request Preparation
→ Cross-Team Human Review
→ Merge Decision
→ Sustainable Project State
```

---

## FW-P01 — Requirement & Architecture Analysis

```text
Final Workshop requirement:

Priority customer escalation workflow'u mevcut
call-center-integration-platform architecture'ına eklemek istiyoruz.

Henüz implementation yapma.

Önce şunları çıkar:

1. existing architecture surface
2. affected modules
3. shared domain contracts
4. integration boundaries
5. security/data concerns
6. likely test boundaries
7. unresolved domain questions
8. explicit non-goals

Evidence ve inference ayrımını koru.
```

---

## FW-P02 — Team Decomposition

```text
Architecture analysis sonucunu iki parallel workstream'e böl:

Squad A:
- IVR
- CRM

Squad B:
- Bot / AI decision boundary
- Notification

Her squad için şunları belirle:

- owned files/modules
- owned behavior
- shared contracts
- dependencies on other squad
- acceptance criteria
- verification responsibility
- Git boundary
- human approval gate

Shared contract değişikliklerini ayrıca "Contract Freeze Gate" altında topla.

Henüz implementation yapma.
```

---

## FW-P03 — Contract Freeze Gate

```text
İki squad'ın ortak kullanacağı Customer / Call Context contract'ını incele.

Contract freeze öncesinde şunları çıkar:

- required fields
- optional fields
- ownership
- nullability/validation assumptions
- backward compatibility
- security/privacy classification
- versioning impact

Açık decision varsa implementation başlamadan human decision iste.

Contract freeze tamamlanmadan squad implementation'ına geçme.
```

---

## FW-P04 — Squad Plan

```text
Yalnız kendi squad scope'un için implementation plan üret.

Plan şu alanları içersin:

- Goal
- Acceptance Criteria
- Change Surface
- Tests First
- Allowed Actions
- Forbidden Actions
- Verification Commands
- Git Boundary
- Stop Conditions

Other squad'ın owned files'ını değiştirme.
```

---

## FW-P05 — Controlled Implementation

```text
Human approval verilen squad planını uygula.

Rules:

- yalnız owned files
- shared contract freeze'e uy
- synthetic data only
- network yok
- production yok
- deployment yok
- Git remote action yok
- unexpected cross-squad dependency çıkarsa STOP

Targeted tests için command'ı önce göster ve approval iste.
```

---

## FW-P06 — Testing & Debugging

```text
Targeted tests'i çalıştır.

Failure varsa:

Failure
→ Evidence
→ Hypothesis
→ Root Cause
→ Minimal Fix
→ Re-run

workflow'unu kullan.

Assertion weakening, test disabling veya unrelated refactor yapma.
```

---

## FW-P07 — Automated Controls

```text
Change için mevcut deterministic controls/hook'ları çalıştır veya
required verification evidence'i üret.

Raporla:

- control
- pass/fail
- what it proves
- what it does not prove

New hook/rule eklemek gerekiyorsa ayrıca approval iste.
```

---

## FW-P08 — Git Diff & Atomic Commit Plan

```text
Git diff'i read-only incele.

Raporla:

- changed files
- owned vs non-owned files
- unrelated changes
- test changes
- architecture impact
- security concerns

Ardından atomic commit boundary ve commit message öner.

Henüz commit/push yapma.
```

---

## FW-P09 — Pull Request Preparation

```text
Squad change'i için PR evidence package hazırla:

- Requirement
- Scope
- Architecture impact
- Files changed
- Tests
- Controls/hooks
- Risks
- Explicit non-changes
- Reviewer questions
- Cross-squad contract impact

Remote PR oluşturma.
```

---

## FW-P10 — Cross-Team Review

```text
Other squad'ın PR evidence package ve diff'ini reviewer olarak değerlendir.

Kontrol et:

- shared contract compliance
- architecture boundary
- acceptance criteria
- tests
- unrelated changes
- data/security
- delivery blockers

Sonuç şu kategorilerden biri olsun:

- Approve
- Request changes
- Needs clarification

Merge yapma.
```

---

## FW-P11 — Human Merge Decision

```text
Final merge-readiness summary üret.

Şunları birleştir:

- both squad reviews
- test evidence
- deterministic controls
- unresolved risks
- contract compatibility
- Git cleanliness
- documentation/knowledge state

Recommendation şu kategorilerden biri olsun:

- Ready
- Ready with concerns
- Not ready

Merge action alma.

Final merge decision human owner'a aittir.
```

---

## FW-P12 — Sustainable Project State

```text
Workshop sonunda repository'nin sustainable state'ini değerlendir.

Kontrol et:

- organizational knowledge repository'de mi?
- temporary prompt-only decisions var mı?
- rules/docs güncel mi?
- vendor-specific detail ile portable principle ayrılmış mı?
- usage/cost için iyileştirme fırsatları var mı?
- open technical debt açıkça kaydedilmiş mi?

Henüz yeni change yapma.
Sadece final state assessment üret.
```

---

## Workshop Completion Criteria

```text
Architecture understood
+ contracts frozen
+ ownership clear
+ bounded implementation
+ tests green
+ deterministic controls passed
+ diff reviewed
+ PR evidence prepared
+ cross-team review complete
+ human merge decision preserved
+ repository knowledge updated appropriately
```

## Final Principle

> **Claude engineering süreçlerini hızlandırır. İnsanlar intent'i tanımlar. İnsanlar boundary'leri approve eder. Automation evidence üretir. İnsanlar review eder ve karar verir. Repository organizational knowledge'i korur.**
