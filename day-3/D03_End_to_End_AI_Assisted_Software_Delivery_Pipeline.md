# D03 — End-to-End AI-Assisted Software Delivery Pipeline

## Öğrenme Hedefi

Tek bir business requirement'ı Claude Code ile kontrollü ve evidence-driven bir software delivery lifecycle boyunca ilerletmek.

Bu lab sonunda katılımcı:

- requirement ile implementation solution'ını birbirinden ayırabilir,
- repository evidence toplamadan technical solution uydurmaz,
- repository baseline ve shared engineering rules'i change öncesinde kontrol eder,
- bounded exploration için doğru yerde subagent kullanır,
- implementation öncesinde açık bir human gate uygular,
- approved plan'i main thread'de minimum change ile implement eder,
- focused tests ve full verification arasındaki farkı uygular,
- `git diff` üzerinden local evidence üretir,
- implementation'dan bağımsız read-only verification yapar,
- review findings için ikinci bir human decision gate uygular,
- atomic commit boundary oluşturur,
- local commit ile remote delivery authority'yi birbirinden ayırır,
- Pull Request için izlenebilir evidence package üretir,
- CI sonucunu bağımsız delivery evidence olarak değerlendirir,
- merge-readiness üretirken final merge accountability'yi insanda tutar.

Amaç yalnızca Claude Code'a bir feature yaptırmak değildir.

Amaç, Claude Code'u kontrollü bir **AI-Assisted Software Delivery Workflow** içinde kullanmaktır.

---

## Ana Fikir

```text
Doğrudan şu akışa gitme:

Business Requirement
        ↓
Implementation
        ↓
Commit
        ↓
Push

Bunun yerine:

Business Requirement
        ↓
Requirement Analysis
        ↓
Repository Evidence
        ↓
Implementation Plan
        ↓
Human Gate
        ↓
Implementation
        ↓
Verification
        ↓
Independent Review
        ↓
Human Gate
        ↓
Targeted Fix
        ↓
Final Verification
        ↓
Commit Boundary
        ↓
Human Gate
        ↓
Local Commit
        ↓
PR Evidence
        ↓
Remote Delivery Gate
        ↓
CI
        ↓
Merge Readiness
        ↓
Human Merge Decision
```

Bu pipeline boyunca dört prensip korunur:

```text
Evidence
≠
Inference
≠
Assumption
≠
Unknown
```

ve:

```text
Local Evidence
≠
Remote Authority
```

ve:

```text
Agent Autonomy
+
Explicit Boundaries
+
Human Gates
+
Independent Verification
```

ve:

```text
Anlama
≠
Değiştirme

Değiştirme
≠
Verification

Verification
≠
Delivery
```

---

## Lab Senaryosu

Bu pipeline, `call-center-integration-platform` repository'si üzerinde çalıştırılacaktır.

### Business Requirement

```text
Notification delivery sırasında geçici bir failure oluşması,
primary incoming call flow'unun başarısız olmasına neden olmamalıdır.

Notification delivery en fazla üç kez denenmelidir.

Üçüncü denemeden sonra delivery hâlâ başarısızsa:

- primary call flow normal şekilde tamamlanabilmeli,
- notification failure observable olmalı,
- failure sessizce swallow edilmemelidir.

Başarılı notification behavior değişmemelidir.
```

Bu requirement bilinçli olarak implementation detail içermez.

Şunları baştan varsayma:

```text
- RetryPolicy class var.
- NotificationService var.
- Notification async çalışıyor.
- Queue var.
- Retry library kullanılıyor.
- Exception tipi belli.
- Logging implementation'ı belli.
- Notification failure şu layer'da handle edilmeli.
```

Repository evidence bunları doğrulamalıdır.

---

## Gerekli Başlangıç Durumu

Bu lab'e başlamadan önce mümkünse aşağıdaki Day 1–3 çalışmaları tamamlanmış olmalıdır:

```text
- repository local olarak mevcut
- project build edilebilir durumda
- tests çalıştırılabilir durumda
- CLAUDE.md / .claude/rules varsa repository'de mevcut
- M10 sırasında oluşturulan integration-flow-explorer subagent mevcut
- M10 sırasında oluşturulan read-only command hook mevcut
- Git repository initialize edilmiş
- Git remote kullanılıyorsa doğru remote configure edilmiş
- GitHub CLI kullanılıyorsa authenticated
```

M10'dan beklenen project-scoped subagent:

```text
.claude/agents/integration-flow-explorer.md
```

M10'dan beklenen hook:

```text
.claude/hooks/validate-readonly-command.ps1
```

Bu artifact'lar mevcut değilse pipeline onları sessizce uydurmamalıdır.

Eksik dependency veya prerequisite varsa Claude bunu evidence gap olarak raporlamalıdır.

---

# Pipeline Genel Akışı

```text
                         SHARED OPERATING MODEL
            CLAUDE.md / Rules / Hooks / Permissions / Git Policy
                                   │
                                   ▼
                        BUSINESS REQUIREMENT
                                   │
                                   ▼
                    P01 — Requirement Analysis
                                   │
                                   ▼
                    P02 — Repository Baseline
                                   │
                                   ▼
              P03 — Evidence-Based Exploration
                                   │
                                   ▼
                    P04 — Implementation Plan
                                   │
                         ┌─────────┴─────────┐
                         │   HUMAN GATE #1   │
                         │  Plan'i Onayla │
                         └─────────┬─────────┘
                                   │
                                   ▼
                     P05 — Implementation
                                   │
                                   ▼
                     P06 — Focused Tests
                                   │
                                   ▼
                 P07 — Full Verification
                                   │
                                   ▼
                    P08 — Git Diff Review
                                   │
                                   ▼
              P09 — Independent Verification
                                   │
                         ┌─────────┴─────────┐
                         │   HUMAN GATE #2   │
                         │ Fix'leri Onayla │
                         └─────────┬─────────┘
                                   │
                                   ▼
                     P10 — Targeted Fixes
                                   │
                                   ▼
                  P11 — Final Verification
                                   │
                                   ▼
                P12 — Commit Boundary Planning
                                   │
                         ┌─────────┴─────────┐
                         │   HUMAN GATE #3   │
                         │  Commit'i Onayla   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                      P13 — Local Commit
                                   │
                                   ▼
                   P14 — PR Evidence Package
                                   │
                                   ▼
                    P15 — Remote Delivery
                                   │
                         ┌─────────┴─────────┐
                         │   HUMAN GATE #4   │
                         │ Push / PR'ı Onayla │
                         └─────────┬─────────┘
                                   │
                                   ▼
                      P16 — CI Validation
                                   │
                                   ▼
                  P17 — Merge Readiness
                                   │
                         ┌─────────┴─────────┐
                         │ FINAL HUMAN GATE  │
                         │   Merge Kararı  │
                         └───────────────────┘
```

---

# PHASE 1 — ANLA

---

## D03-CAP-P01 — Business Requirement'ı Analiz Et

### Mode

```text
Plan Mode
```

### Amaç

Repository'ye dokunmadan önce business requirement'ın:

- ne istediğini,
- neyi değiştirmemesi gerektiğini,
- hangi teknik soruları açık bıraktığını,
- hangi acceptance criteria'nın doğrudan requirement'tan türetilebildiğini

ayırmak.

### Prompt

```text
Yeni business requirement:

"Notification delivery sırasında geçici bir failure oluşması,
primary incoming call flow'unun başarısız olmasına neden olmamalıdır.

Notification delivery en fazla üç kez denenmelidir.

Üçüncü denemeden sonra delivery hâlâ başarısızsa:

- primary call flow normal şekilde tamamlanabilmeli,
- notification failure observable olmalı,
- failure sessizce swallow edilmemelidir.

Başarılı notification behavior değişmemelidir."

Bu aşamada repository exploration yapma.

Henüz hiçbir source file okuma.
Code edit etme.
Test çalıştırma.
Build çalıştırma.
Git action alma.
Implementation plan üretme.

Yalnızca business requirement'ı analiz et.

Aşağıdaki başlıklarla cevap ver:

## İstenen Behavior

Requirement'ın açıkça istediği behavioral changes'i çıkar.

## Değişmemesi Gerekenler

Requirement'a göre korunması gereken behavior'ları çıkar.

## Açık Constraints

Requirement içerisinde açıkça belirtilmiş constraint'leri çıkar.

## Unknowns

Repository evidence olmadan cevaplanamayacak teknik soruları listele.

Özellikle:

- notification execution boundary
- sync vs async behavior
- existing retry mechanism
- exception types
- existing error-handling convention
- logging / observability mechanism
- current test coverage
- primary call flow ile notification coupling'i

değerlendir.

## Yapılmaması Gereken Assumptions

Repository'yi görmeden yapılmaması gereken assumption'ları listele.

## Acceptance Criteria

Yalnız requirement'tan doğrudan türetilebilen observable acceptance criteria üret.

Repository'de varlığı kanıtlanmamış class, method, framework veya abstraction isimleri kullanma.

## Repository Exploration Soruları

Bir sonraki exploration aşamasında evidence ile cevaplanması gereken soruları priority order ile listele.

Evidence olmayan hiçbir technical detail'i gerçekmiş gibi sunma.

Implementation veya solution önermeden dur.
```

### Neleri Gözlemlemeli?

- Claude implementation'a erken atlıyor mu?
- Repository'yi okumaya çalışıyor mu?
- Requirement ile solution design'ı karıştırıyor mu?
- Unknown ve assumption'ları gerçekten ayırıyor mu?
- Acceptance criteria behavioral mı?
- Var olmayan abstraction isimleri uyduruluyor mu?
- Başarılı path'in korunması explicit constraint olarak yakalanıyor mu?

### Tartışma Noktası

```text
Requirement Analysis
≠
Solution Design
```

> **Repository evidence olmadan verilen implementation kararı plan değil, assumption'dır.**

### Durma Koşulu

Bu prompt sonunda hiçbir tool-based repository exploration başlamamalıdır.

---

## D03-CAP-P02 — Repository Baseline'i Belirle

### Mode

```text
Manual Mode
```

### Amaç

Feature exploration başlamadan önce repository'nin güvenli başlangıç state'ini belirlemek.

### Prompt

```text
Şimdi repository baseline'i belirle.

Henüz business requirement için implementation exploration yapma.
Code edit etme.
Dependency değiştirme.
Git state mutation yapma.

Önce repository-level instructions ve mevcut local state'i incele.

İncele:

- CLAUDE.md
- .claude/rules/*
- relevant project documentation
- build system
- test structure
- current branch
- working tree state
- mevcut modified / untracked files
- repository'nin build/test için kullandığı command'lar

Git için yalnız read-only inspection kullan:

- git status
- git diff
- git diff --stat
- git log gerekirse

Mutating Git command kullanma.

Özellikle raporla:

## Shared Engineering Rules

Bu task'i etkileyen architecture, testing, security/data ve Git rules.

## Build ve Test Baseline

Build tool, test tool ve relevant commands.

## Git Baseline

- current branch
- working tree clean mi
- pre-existing changes var mı
- change isolation açısından risk var mı

## Pipeline Prerequisites

Kontrol et:

- .claude/agents/integration-flow-explorer.md mevcut mu?
- .claude/hooks/validate-readonly-command.ps1 mevcut mu?
- Git remote mevcut mu?
- CI configuration mevcut mu?

Eksik prerequisite varsa bunu açıkça belirt.

## Baseline Kararı

Şunlardan birini seç:

- SAFE TO CONTINUE
- CONTINUE WITH CONCERNS
- STOP — BASELINE NOT SAFE

Pre-existing changes varsa bunları bizim feature change'imizmiş gibi sahiplenme.

Bu aşamada business requirement için solution önermeden dur.
```

### Neleri Gözlemlemeli?

- Claude existing user changes'i kendi task'ına dahil ediyor mu?
- Shared rules gerçekten okunuyor mu?
- Build/test command'ları evidence'a mı dayanıyor?
- Git mutation yapılıyor mu?
- Prerequisite eksikleri uydurulmadan raporlanıyor mu?

### Tartışma Noktası

```text
Sistemi değiştirmeden önce,
sistemin mevcut state'ini bil.
```

> **Sağlıklı reasoning, dirty working tree üzerinde başlatılmamalıdır.**

### Human Kontrolü

`STOP — BASELINE NOT SAFE` çıkarsa pipeline'a devam etme.

Önce baseline problemi human olarak çöz.

---

## D03-CAP-P03 — Notification Flow'u Custom Subagent ile Explore Et

### Mode

```text
Plan Mode
```

### Amaç

Main thread context'ini repository exploration detaylarıyla şişirmeden, bounded ve evidence-based bir change surface çıkarmak.

### Prompt

```text
Business requirement:

"Notification delivery sırasında geçici bir failure oluşması,
primary incoming call flow'unun başarısız olmasına neden olmamalıdır.

Notification delivery en fazla üç kez denenmelidir.

Üçüncü denemeden sonra delivery hâlâ başarısızsa:

- primary call flow normal şekilde tamamlanabilmeli,
- notification failure observable olmalı,
- failure sessizce swallow edilmemelidir.

Başarılı notification behavior değişmemelidir."

Bu aşamada code edit etme.

Yalnızca project-scoped integration-flow-explorer subagent'ını kullanarak
repository'yi araştır.

Subagent aşağıdaki soruları evidence ile cevaplasın:

1. Incoming call için current execution path nedir?
2. Notification delivery hangi class/method üzerinden başlıyor?
3. Notification domain port/interface veya adapter boundary nedir?
4. Notification delivery sync mi async mı?
5. Current failure behavior nedir?
6. Notification failure primary call flow'u bugün etkiliyor mu?
7. Existing retry abstraction veya retry convention var mı?
8. Existing exception types nelerdir?
9. Existing logging / observability convention nedir?
10. Benzer transient failure handling başka integration boundary'de mevcut mu?
11. Hangi production files minimum change surface'e giriyor?
12. Hangi tests current behavior'ı cover ediyor?
13. Requirement ile repository behavior arasında mismatch var mı?
14. En küçük güvenli implementation direction nedir?

Scope:

- repository exploration
- Read / Grep / Glob
- gerekirse yalnız read-only Bash veya PowerShell inspection
- git status / git diff gibi read-only Git inspection
- implementation yok
- edits yok
- dependency ekleme yok
- build/test yok
- mutating Git yok

Main thread'e yalnızca şu structured summary dönsün:

## Mevcut Flow
## Notification Boundary
## Mevcut Failure Path
## Mevcut Retry / Error-Handling Evidence
## Observability Evidence
## Etkilenen Files
## İlgili Tests
## Minimum Change Yönü
## Risks
## Evidence Gaps

Her önemli conclusion için exact file path ve symbol ver.

Evidence, Inference ve Unknown ayrımını açıkça yap.

Subagent tamamlandıktan sonra main thread hiçbir implementation action almasın.
```

### Neleri Gözlemlemeli?

- Exploration subagent context'inde mi kaldı?
- Main thread'e bounded summary mi döndü?
- Exact file path ve symbol var mı?
- Retry abstraction uyduruldu mu?
- Sync/async behavior evidence ile mi belirlendi?
- Scope notification requirement çevresinde kaldı mı?
- Gereksiz architecture refactor açıldı mı?

### Tartışma Noktası

```text
Subagent
→ explore et ve uncertainty'yi azalt

Main Thread
→ karar ver ve integrate et
```

> **Exploration'ın amacı code yazmak değil, implementation öncesindeki uncertainty'yi azaltmaktır.**

### Durma Koşulu

Subagent report geldikten sonra implementation başlamamalıdır.

---

## D03-CAP-P04 — Evidence-Based Implementation Plan Üret

### Mode

```text
Plan Mode
```

### Amaç

P01 requirement analysis ile P03 repository evidence'ını birleştirip bounded implementation plan üretmek.

### Prompt

```text
P01 requirement analysis ve
integration-flow-explorer tarafından dönen repository evidence'ını kullan.

Henüz code edit etme.
Test çalıştırma.
Build çalıştırma.
Git mutation yapma.

Bir implementation plan üret.

Plan şu başlıklarda olsun:

## Evidence Özeti

Yalnız repository'de doğrulanmış önemli facts.

## Requirement → Evidence Mapping

Her acceptance criterion için ilgili current code/test evidence'ını eşleştir.

## Önerilen Change Surface

Değişmesi gereken exact files ve symbols.

Her biri için neden gerekli olduğunu açıkla.

## Explicit Non-Changes

Bu task sırasında değiştirilmemesi gereken önemli boundaries.

## Implementation Adımları

Minimum ve sequential bir change plan.

Her step bir engineering reason taşısın.

## Test Plan

Ayır:

- focused tests
- regression-relevant tests
- full verification

## Observability Plan

Failure'ın nasıl observable kalacağını existing repository convention'a göre açıkla.

Existing convention yoksa uydurma; bunu decision gap olarak belirt.

## Risks

Özellikle:

- exception swallowing
- retry amplification
- duplicate side effects
- blocking behavior
- timing impact
- coupling
- regression on successful path

değerlendir.

## Açık Decisions

Human approval gerektiren unresolved decision varsa açıkça listele.

Unrelated refactor önermeme.
Yeni dependency yalnız repository evidence gerçekten gerektiriyorsa öner.
Framework veya abstraction uydurma.

Plan sonunda implementation yapmadan dur.
```

### Neleri Gözlemlemeli?

- Plan evidence'a bağlı mı?
- Exact change surface bounded mı?
- New abstraction gereksiz yere ekleniyor mu?
- Retry duplicate side-effect riski düşünülüyor mu?
- Başarılı path için regression test var mı?
- Observability existing convention'a dayanıyor mu?
- Open decisions saklanıyor mu yoksa Claude kendi kapatıyor mu?

### Tartışma Noktası

```text
İyi Plan
=
Requirement
+
Repository Evidence
+
Minimum Change
+
Verification Strategy
```

---

# HUMAN GATE #1 — PLAN'I ONAYLA

Pipeline burada durur.

Human reviewer olarak kontrol et:

```text
- Requirement doğru anlaşılmış mı?
- Evidence yeterli mi?
- Change surface minimum mu?
- Architecture boundary doğru mu?
- Retry doğru layer'da mı?
- Side-effect duplication riski var mı?
- Başarılı behavior korunuyor mu?
- Test plan yeterli mi?
- Unresolved decision var mı?
```

Plan uygun değilse Claude'a planı revize ettir.

Implementation'a ancak açık human approval sonrasında geç.

---

# PHASE 2 — DEĞİŞTİR

---

## D03-CAP-P05 — Approved Plan'i Main Thread'de Implement Et

### Mode

```text
Default Mode
```

### Amaç

Approved plan'i minimum change ile implementation'a dönüştürmek.

### Prompt

```text
P04 implementation plan human review'dan geçti ve onaylandı.

Şimdi implementation'ı main thread'de yap.

Bu task için implementation subagent kullanma.

Yalnız approved plan ve repository evidence'a dayan.

Edit öncesinde kısaca raporla:

1. değiştirilecek exact files
2. her file'ın neden değişeceği
3. korunacak önemli boundaries
4. beklenen behavioral change

Sonra implementation yap.

Constraints:

- successful notification behavior değişmemeli
- yalnız required transient notification failure behavior handle edilmeli
- primary call flow notification failure nedeniyle fail olmamalı
- retry sayısı requirement'taki limit ile uyumlu olmalı
- existing domain/integration abstraction mümkün olduğunca reuse edilmeli
- exception scope gereksiz genişletilmemeli
- unrelated refactor yapılmamalı
- approved plan açıkça gerektirmedikçe yeni dependency eklenmemeli
- logging/observability existing repository convention'a uymalı
- focused tests eklenmeli veya mevcut tests güncellenmeli

Eğer implementation sırasında approved plan'i geçersiz kılan yeni repository evidence bulunursa:

- uydurarak devam etme
- değişiklikleri genişletme
- critical mismatch'i raporla
- execution'ı durdur

Implementation tamamlandığında henüz commit/push yapma.

Özetle:

- files changed
- behavioral change
- eklenen/güncellenen tests
- plan deviation varsa
- remaining uncertainty varsa

raporla.
```

### Neleri Gözlemlemeli?

- Main thread sequential decision chain'i koruyor mu?
- Plan dışına çıkılıyor mu?
- Unrelated refactor yapılıyor mu?
- Exception catch çok geniş mi?
- Retry side-effect safe mi?
- Başarılı path değişiyor mu?
- Implementation subagent'a delege edildi mi?

### Tartışma Noktası

```text
Exploration'ı isolated yap.
Implementation ownership'i görünür tut.
```

---

## D03-CAP-P06 — Önce Focused Tests Çalıştır

### Mode

```text
Default Mode
```

### Amaç

Önce changed behavior'a en yakın feedback loop'u çalıştırmak.

### Prompt

```text
Implementation tamamlandı.

Henüz full test suite çalıştırma.

Önce yalnız changed behavior'a en yakın focused tests'i belirle.

Çalıştırmadan önce:

- hangi test command'ını çalıştıracağını
- hangi behavior'ı verify ettiğini

açıkla.

Sonra focused tests'i çalıştır.

Raporla:

## Focused Test Command
## Çalıştırılan Tests
## Sonuç
## Failure Detayları
## Requirement Coverage

Özellikle şu acceptance behavior'ları doğrula:

- başarılı notification path korunuyor
- transient failure retry ediliyor
- retry limit aşılmıyor
- final notification failure primary call flow'u fail ettirmiyor
- failure observable kalıyor

Focused tests fail ederse:

- hemen geniş refactor yapma
- önce root cause'u sınıflandır
- test defect mi
- implementation defect mi
- requirement mismatch mi
- environment issue mu

belirt ve dur.

Commit/push yapma.
```

### Neleri Gözlemlemeli?

- Full test suite'e gereksiz erken gidiliyor mu?
- Failure olduğunda hemen random edit yapılıyor mu?
- Test gerçekten requirement behavior'ını ölçüyor mu?
- Test sadece implementation detail'e mi bağlı?

### Tartışma Noktası

```text
Önce hızlı feedback al.
Sonra daha geniş confidence oluştur.
```

---

## D03-CAP-P07 — Full Verification Çalıştır

### Mode

```text
Default Mode
```

### Amaç

Focused correctness sonrasında regression riskini daha geniş scope'ta kontrol etmek.

### Prompt

```text
Focused tests geçti.

Şimdi repository'nin documented verification workflow'una göre
relevant full verification çalıştır.

Repository evidence'a göre uygun olanları kullan:

- full unit test suite
- integration tests
- build
- static checks
- repository-defined verification commands

Command'ları uydurma.
P02 baseline'da doğrulanan build/test workflow'unu kullan.

Her command için:

- command
- purpose
- result
- tool output'ta duration bilgisi varsa
- failure varsa exact failure evidence

raporla.

Bir failure çıkarsa:

1. feature change ile ilişkili mi belirle
2. pre-existing failure ihtimalini baseline evidence ile karşılaştır
3. root cause doğrulanmadan fix yapma

Bu aşamada commit/push yapma.
```

### Neleri Gözlemlemeli?

- P02 baseline evidence kullanılıyor mu?
- Pre-existing failure ile new regression ayrılıyor mu?
- Full suite failure otomatik olarak bizim change'e bağlanıyor mu?
- Verification sonucu açık evidence olarak tutuluyor mu?

### Tartışma Noktası

```text
Test passing
≠
Change understood

but

Test evidence
+
Diff evidence
+
Review evidence
→ stronger confidence
```

---

## D03-CAP-P08 — Git Diff'i İncele

### Mode

```text
Manual Mode
```

### Amaç

Implementation'ı intention üzerinden değil actual diff üzerinden değerlendirmek.

### Prompt

```text
Şimdi repository'nin local Git state'ini read-only olarak incele.

Allowed:

- git status
- git diff
- git diff --stat
- gerekirse read-only git log

Not allowed:

- git add
- git commit
- git push
- git pull
- git merge
- git rebase
- git reset
- git checkout/switch mutation
- branch create/delete
- remote mutation

Rapor:

## Working Tree State
## Değişen Files
## Diff Özeti
## Beklenen Changes
## Beklenmeyen Changes
## Unrelated Changes
## Değişen Test Files
## Documentation / Config Changes
## Diff Risk Değerlendirmesi

Her changed file için:

- neden değişti
- approved plan ile ilişkisi
- unrelated change olup olmadığı

belirt.

Eğer unexpected veya unrelated diff varsa otomatik düzeltme yapma.

Yalnız evidence üret ve dur.
```

### Neleri Gözlemlemeli?

- Claude source code intention'ını mı anlatıyor, actual diff'i mi?
- Unrelated changes işaretleniyor mu?
- Git mutation yapılıyor mu?
- Diff approved plan ile karşılaştırılıyor mu?

### Tartışma Noktası

```text
Implementation anlatımı
evidence değildir.

Asıl evidence diff'tir.
```

---

# PHASE 3 — DOĞRULA

---

## D03-CAP-P09 — Independent Read-Only Verification Yap

### Mode

```text
Plan Mode
```

### Amaç

Implementation'ı yapan main thread'den bağımsız architectural ve behavioral review almak.

### Prompt

```text
Implementation, focused tests, full verification ve git diff hazır.

Şimdi integration-flow-explorer subagent'ını tekrar kullan.

Amaç:

Mevcut git diff ve etkilenen execution path üzerinden implementation'ı
read-only olarak verify etmek.

Hiçbir file değiştirme.
Hiçbir fix uygulama.
Test değiştirme.
Commit/push yapma.

Aşağıdaki criteria'yı tek tek değerlendir:

1. Business requirement gerçekten karşılanıyor mu?
2. Başarılı notification path korunmuş mu?
3. Primary call flow notification final failure'dan izole mi?
4. Retry behavior requirement limit'i ile uyumlu mu?
5. Retry doğru architectural boundary'de mi?
6. Existing abstractions reuse edilmiş mi?
7. Exception handling beklenenden daha geniş exception'ları swallow ediyor mu?
8. Duplicate side-effect riski oluşuyor mu?
9. Failure observable kalıyor mu?
10. Unnecessary coupling oluşmuş mu?
11. Unrelated refactor var mı?
12. Tests failure path'i gerçekten doğruluyor mu?
13. Regression riski var mı?
14. Implementation approved plan ile uyumlu mu?

Her criterion için yalnız:

- PASS
- CONCERN
- FAIL

status'larından birini kullan.

Her finding için:

- exact file path
- symbol/method
- evidence
- rationale
- severity: low / medium / high

ver.

Final output:

## Verification Özeti
## PASS
## CONCERNS
## FAILURES
## Önerilen Human Decisions

Subagent hiçbir code change yapmasın.
```

### Neleri Gözlemlemeli?

- Review gerçekten read-only mi?
- Aynı agent role boundary korunuyor mu?
- Automatic fix yapılıyor mu?
- Findings evidence ile mi destekleniyor?
- Severity gereksiz büyütülüyor mu?
- Reviewer approved plan'i referans alıyor mu?

### Tartışma Noktası

```text
Implementation
→ sistemi değiştirebildiğimizi gösterir

Independent verification
→ güvenli değiştirip değiştirmediğimizi sorgular
```

---

# HUMAN GATE #2 — FIX'LERİ ONAYLA

P09 sonucunu human reviewer olarak değerlendir.

Her finding için:

```text
- accept
- reject
- defer
```

kararı ver.

Claude'un bulduğu her finding otomatik olarak doğru değildir.

Özellikle:

```text
HIGH
→ evidence var mı?

MEDIUM
→ requirement ile gerçekten ilişkili mi?

LOW
→ fix cost / scope expansion yaratıyor mu?
```

değerlendir.

Yalnız approved findings fix scope'una girmelidir.

---

## D03-CAP-P10 — Yalnız Approved Findings'i Uygula

### Mode

```text
Default Mode
```

### Prompt

```text
Independent verification tamamlandı.

Human review sonucunda yalnız aşağıdaki findings approve edildi:

[APPROVED FINDINGS BURAYA YAPIŞTIR]

Yalnız bu findings için targeted fix uygula.

Not allowed:

- rejected finding'i değiştirmek
- deferred finding'i değiştirmek
- unrelated refactor
- opportunistic cleanup
- new dependency
- architecture redesign
- Git commit/push

Her fix öncesinde:

- finding
- exact file/symbol
- intended minimal change

belirt.

Fix sonrası:

- files changed
- behavior changed
- tests affected
- approved finding'in nasıl çözüldüğü

raporla.
```

### Neleri Gözlemlemeli?

- Scope creep var mı?
- Low-value cleanup yapılıyor mu?
- Rejected/deferred finding'e dokunuluyor mu?
- Human decision gerçekten execution boundary oluyor mu?

### Tartışma Noktası

```text
AI review finding
≠
human tarafından approved engineering work
```

---

## D03-CAP-P11 — Final Verification

### Mode

```text
Default Mode
```

### Amaç

Fix sonrası final local evidence package üretmek.

### Prompt

```text
Targeted fixes tamamlandı.

Şimdi final local verification yap.

Sırasıyla:

1. etkilenen focused tests
2. ilgili full test suite
3. build / repository-defined verification
4. git status
5. git diff
6. git diff --stat

çalıştır.

Git tarafında yalnız read-only command kullan.

Rapor:

## Focused Test Sonucu
## Full Verification Sonucu
## Build Sonucu
## Final Değişen Files
## Final Diff Özeti
## Requirement Coverage
## Kalan Risks
## Bilinen Unknowns

Ayrıca şu mapping'i oluştur:

Requirement
→ Acceptance Criterion
→ Code Change
→ Test Evidence

Eksik mapping varsa işaretle.

Bu aşamada git add, commit veya push yapma.
```

### Neleri Gözlemlemeli?

- Final verification fix sonrası tekrarlandı mı?
- Traceability gap var mı?
- Remaining risk saklanıyor mu?
- Commit'e erken geçiliyor mu?

### Tartışma Noktası

```text
Coding tamamlandı
≠
delivery için hazır
```

---

# PHASE 4 — LOCAL DELIVERY

---

## D03-CAP-P12 — Atomic Commit Boundary'yi Planla

### Mode

```text
Manual Mode
```

### Prompt

```text
Final local verification geçti.

Mevcut diff'i kullanarak atomic commit boundary öner.

Henüz:

- git add
- git commit
- git push

yapma.

Rapor:

## Commit Amacı
## Dahil Edilecek Files
## Hariç Tutulacak Files
## Neden Atomic?
## Önerilen Commit Message
## Verification Evidence
## Bilinen Risks
## Açıkça Yasak Remote Actions

Eğer diff içinde unrelated changes varsa:

- ayrı göster
- bu commit'e dahil etme
- otomatik split veya cleanup yapma

Commit message repository convention varsa ona uysun.

Sonunda human approval bekle.
```

### Neleri Gözlemlemeli?

- One commit = one coherent engineering reason prensibi korunuyor mu?
- Unrelated file commit'e dahil ediliyor mu?
- Agent approval almadan staging yapıyor mu?

### Tartışma Noktası

```text
One commit
→ tek bir tutarlı engineering reason
```

---

# HUMAN GATE #3 — LOCAL COMMIT'İ ONAYLA

Human reviewer kontrolü:

```text
- included files doğru mu?
- excluded files doğru mu?
- tests geçti mi?
- diff temiz mi?
- commit message doğru mu?
- unrelated change var mı?
```

Uygunsa açık approval ver.

---

## D03-CAP-P13 — Local Commit Oluştur

### Mode

```text
Manual Mode
```

### Prompt

```text
Human approval verildi.

Yalnız approved commit boundary içindeki files için local commit oluştur.

Önce execution planını göster:

- stage edilecek files
- commit message
- remote action olmayacağını belirt

Sonra yalnız approved local commit işlemlerini yap.

Not allowed:

- git push
- git pull
- remote branch mutation
- PR creation
- merge
- rebase
- reset
- unrelated files'i stage etmek

Commit sonrası read-only olarak raporla:

- commit hash
- commit message
- committed files
- remaining working tree state
- uncommitted files varsa

Remote action alma ve dur.
```

### Neleri Gözlemlemeli?

- Approved boundary dışı file stage ediliyor mu?
- Local commit sonrası otomatik push oluyor mu?
- Remaining working tree doğru raporlanıyor mu?

### Tartışma Noktası

```text
Local Commit
≠
Remote Delivery Authority
```

---

# PHASE 5 — REMOTE DELIVERY

---

## D03-CAP-P14 — Pull Request Evidence Package Hazırla

### Mode

```text
Manual Mode
```

### Prompt

```text
Local commit hazır.

Henüz GitHub'da PR oluşturma.
Push yapma.
Remote action alma.

Mevcut local evidence üzerinden Pull Request evidence package hazırla.

Şunları üret:

## Problem / Requirement
## Scope
## Acceptance Criteria
## Architecture Etkisi
## Değişen Files
## Değişen Behavior
## Explicit Non-Changes
## Çalıştırılan Tests
## Test Sonuçları
## Risks / Trade-offs
## Reviewer Soruları
## Rollback / Recovery Notu

Sonra traceability mapping üret:

Requirement
→ Acceptance Criterion
→ Code Change
→ Test Evidence

Her claim'i local evidence'a bağla.

Bilmediğin şeyi uydurma.

Son olarak merge-readiness değil,
yalnız PR evidence completeness değerlendir:

- COMPLETE
- COMPLETE WITH GAPS
- INCOMPLETE

Remote action yapmadan dur.
```

### Neleri Gözlemlemeli?

- PR description implementation diary'ye dönüşüyor mu?
- Reviewer'ın karar vereceği evidence var mı?
- Requirement→test traceability açık mı?
- Unsupported claim var mı?

### Tartışma Noktası

```text
Pull Request
yalnızca bir delivery mechanism değildir.

Aynı zamanda bir review contract'tır.
```

---

## D03-CAP-P15 — Remote Delivery Action'ı Hazırla

### Mode

```text
Manual Mode
```

### Amaç

Remote authority kullanılmadan önce exact remote action'ı görünür hale getirmek.

### Prompt

```text
PR evidence package hazır.

Henüz push veya PR create etme.

Önce remote delivery capability ve target'ı read-only olarak doğrula.

Kontrol et:

- current branch
- configured remote
- target base branch
- GitHub CLI availability varsa
- authentication state doğrulanabiliyorsa
- existing PR var mı
- CI workflow repository'de mevcut mu

Rapor:

## Remote Target
## Source Branch
## Base Branch
## Mevcut PR State
## CI Durumu
## Önerilen Remote Actions

Önerilen Remote Actions bölümünde tam olarak hangi action'ların alınacağını yaz.

Örneğin gerekiyorsa:

1. current branch'i push et
2. Pull Request oluştur
3. hazırlanan PR title/body'yi kullan

Ama henüz hiçbir remote mutation yapma.

Remote target eksik veya ambiguous ise STOP de.

Human approval bekle.
```

### Neleri Gözlemlemeli?

- Remote target uyduruluyor mu?
- `main` otomatik olarak base branch kabul ediliyor mu?
- Existing PR kontrol ediliyor mu?
- CI varmış gibi davranılıyor mu?

---

# HUMAN GATE #4 — PUSH / PR CREATION'I ONAYLA

Human reviewer olarak kontrol et:

```text
- source branch doğru mu?
- base branch doğru mu?
- remote doğru mu?
- PR evidence hazır mı?
- local tests/build geçti mi?
- secrets veya sensitive data riski var mı?
```

Uygunsa explicit remote delivery approval ver.

---

## D03-CAP-P16 — Push Yap ve Pull Request Oluştur

### Mode

```text
Manual Mode
```

### Prompt

```text
Human tarafından remote delivery approval verildi.

Yalnız approved remote actions'ı uygula.

Allowed:

- approved branch'i approved remote'a push etmek
- prepared evidence package ile Pull Request oluşturmak

Not allowed:

- merge
- force push
- rebase
- unrelated branch mutation
- tag/release oluşturmak
- repository settings değiştirmek

PR oluşturulduktan sonra raporla:

## Remote Branch
## Pull Request
## Base Branch
## PR Title
## Tespit Edilen CI Checks
## Yapılan Remote Actions

Merge yapma.

CI sonucu henüz yoksa bunu açıkça belirt.
```

### Neleri Gözlemlemeli?

- Approved action dışında remote mutation var mı?
- Force push gibi destructive action deneniyor mu?
- PR body P14 evidence'ına dayanıyor mu?
- Merge'e otomatik geçiliyor mu?

### Tartışma Noktası

```text
Push / PR creation
açık authority gerektirir.

Merge için ayrı bir decision gerekir.
```

---

# PHASE 6 — CI & MERGE READINESS

---

## D03-CAP-P17 — CI ve Merge Readiness'i Değerlendir

### Mode

```text
Manual Mode
```

### Prompt

```text
Pull Request oluşturuldu.

Şimdi mevcut PR ve CI state'ini read-only olarak değerlendir.

Remote state'i değiştirme.
Merge yapma.
Workflow re-run veya approval action alma.

Kontrol et:

- CI checks
- build result
- test result
- static analysis sonucu varsa
- review comments varsa
- unresolved conversations varsa
- branch protection / required checks görünüyorsa
- merge conflicts varsa

Önce CI state'i raporla:

## CI Status
## Passing Checks
## Failing Checks
## Pending Checks
## Eksik Beklenen Checks

CI fail oluyorsa:

- root cause evidence'ını özetle
- code defect / test defect / environment / CI configuration olarak sınıflandır
- fix uygulama
- pipeline'ın hangi local stage'ine geri dönülmesi gerektiğini öner

CI geçiyorsa merge-readiness değerlendir.

Aşağıdaki kategorilerden yalnız birini kullan:

- READY
- READY WITH CONCERNS
- NOT READY

Gerekçeyi şu evidence ile açıkla:

- requirement coverage
- local tests
- CI
- diff
- architecture boundary
- unrelated changes
- security/data concerns
- unresolved questions
- review blockers

Final merge decision human reviewer'a aittir.

Merge action alma.
```

### Neleri Gözlemlemeli?

- CI sonucu independent evidence olarak kullanılıyor mu?
- CI fail → otomatik random fix davranışı var mı?
- Merge readiness evidence-driven mı?
- AI merge accountability'yi üstleniyor mu?

### Tartışma Noktası

```text
AI merge evidence hazırlayabilir.

Merge accountability AI'a ait değildir.
```

---

# FINAL HUMAN GATE — MERGE KARARI

Pipeline'ın automation kısmı burada biter.

Human reviewer son kez değerlendirir:

```text
Requirement
        ↓
Acceptance Criteria
        ↓
Repository Evidence
        ↓
Approved Plan
        ↓
Implementation
        ↓
Tests
        ↓
Diff
        ↓
Independent Review
        ↓
Approved Fixes
        ↓
Final Verification
        ↓
Atomic Commit
        ↓
PR Evidence
        ↓
CI
        ↓
Merge Readiness
        ↓
HUMAN MERGE DECISION
```

Final decision:

```text
MERGE
DO NOT MERGE
REQUEST CHANGES
```

Bu lab içerisinde Claude merge action almamalıdır.

---

# Failure Routing Rehberi

Pipeline sırasında failure olduğunda en yakın doğru stage'e geri dön.

```text
Requirement ambiguous ise
→ P01

Repository baseline güvenli değilse
→ P02

Architectural evidence eksikse
→ P03

Plan reddedildiyse
→ P04

Implementation mismatch varsa
→ P05

Focused test fail olursa
→ P06 / P05

Regression failure varsa
→ P07 / P05'e dönmeden önce diagnose et

Unexpected diff varsa
→ P08 / P05

Independent review concern varsa
→ Human Gate #2

Approved review fix varsa
→ P10

Final verification fail olursa
→ P11 → ilgili implementation stage

Commit boundary problemi varsa
→ P12

Remote target belirsizse
→ P15

CI fail olursa
→ P17 diagnosis
→ sonra doğru local stage'e dön
```

Pipeline'ı her failure'da en baştan tekrar çalıştırmak gerekmez.

```text
En başa otomatik dönme.
Geçersiz hale gelen en erken assumption'ın
olduğu stage'e geri dön.
```

---

# Eğitmen Gözlem Checklist'i

Lab boyunca öğrencilerde aşağıdaki davranışları gözlemle:

```text
[ ] Requirement ile implementation'ı ayırıyorlar mı?
[ ] Evidence olmadan class/abstraction uyduruyorlar mı?
[ ] Plan Mode'u doğru aşamada kullanıyorlar mı?
[ ] Subagent'ı bounded exploration için mi kullanıyorlar?
[ ] Main thread implementation ownership'i görünür mü?
[ ] Human Gate gerçekten execution'ı durduruyor mu?
[ ] Focused test → full verification sırası korunuyor mu?
[ ] Diff gerçek evidence olarak inceleniyor mu?
[ ] Independent reviewer otomatik fix yapıyor mu?
[ ] Review finding ile approved work ayrılıyor mu?
[ ] Git read-only ve mutation actions ayrılıyor mu?
[ ] Local commit ile remote authority ayrılıyor mu?
[ ] PR evidence traceable mı?
[ ] CI independent verification olarak görülüyor mu?
[ ] Merge accountability human reviewer'da mı?
```

---

# End-to-End Mental Model

```text
                ANLA
                    │
                    ▼
Requirement → Baseline → Explore → Plan
                              │
                        HUMAN GATE
                              │
                              ▼

                  DEĞİŞTİR
                    │
                    ▼
Implementation → Focused Tests → Full Verification
                              │
                              ▼

                  İNCELE
                    │
                    ▼
                  Git Diff
                    │
                    ▼
            Independent Review
                    │
               HUMAN GATE
                    │
                    ▼

                   DÜZELT
                    │
                    ▼
            Targeted Fixes
                    │
                    ▼
          Final Verification
                    │
                    ▼

              LOCAL DELIVERY
                    │
                    ▼
            Commit Boundary
                    │
               HUMAN GATE
                    │
                    ▼
              Local Commit
                    │
                    ▼

             REMOTE DELIVERY
                    │
                    ▼
             PR Evidence
                    │
                    ▼
             Remote Plan
                    │
               HUMAN GATE
                    │
                    ▼
               Push + PR
                    │
                    ▼

                 DOĞRULA
                    │
                    ▼
                   CI
                    │
                    ▼
            Merge Readiness
                    │
              HUMAN KARARI
```

---

# Final Çıkarım

```text
İyi bir agentic delivery
şu değildir:

prompt
→ code
→ commit
→ push

Şudur:

requirement
→ evidence
→ plan
→ human decision
→ bounded execution
→ verification
→ independent review
→ human decision
→ izlenebilir delivery evidence
→ CI
→ human accountability
```

> **Claude Code, yazılım teslim yaşam döngüsünün tamamında yer alabilir; ancak mühendislik yetkisi açık, sınırları belirlenmiş ve kanıta dayalı kalmalıdır.**
