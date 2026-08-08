# X0 Agents Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Добавить 6 новых агентов из X0-Framework (merge промпт+манифест → один файл) и обогатить 3–4 существующих, по спеке `docs/specs/2026-08-08-x0-agents-expansion-design.md`.

**Architecture:** Каждый агент — один самодостаточный markdown-файл в корне репо (frontmatter + тело), без внешних ссылок на X0-файлы. Формат-эталон — существующий `database-engineer.md` (он уже сделан по этой же схеме merge). Пути к документации — только канонические `docs/`-пути.

**Tech Stack:** Markdown-файлы агентов Claude Code (frontmatter: name, description, model, color, tools, mcpServers). Никакого кода — «тестом» каждой задачи служат grep-проверки чистоты.

## Global Constraints

Применяются к КАЖДОЙ задаче. Источник: спека, секция «Принципы мерджа и чистоты».

1. **Формат-эталон:** перед написанием агента прочитай `database-engineer.md` целиком — структура секций, стиль (микс RU/EN, описания на русском), тон.
2. **Ноль ссылок на X0:** в итоговом файле запрещены: `{{MANIFESTS_ROOT}}`, `{{AGENTS_ROOT}}`, `{{DOCS_ROOT}}`, `{{BACKLOG_ROOT}}`, фразы «см. манифест», «Detailed Manifest», ссылки на `agents/configs/…`, tree metaphor, sandbox-профили, `@multi-agent-specialist`.
3. **Канонические пути** (всегда с оговоркой «если есть в проекте»):

   | Путь | Назначение |
   |---|---|
   | `CLAUDE.md` | стек, архитектура, deployment |
   | `DESIGN.md` (корень) или `docs/design-system.md` | дизайн-система |
   | `docs/ADR/` | архитектурные решения |
   | `docs/conventions/` (`git.md`, `testing.md`, …) | конвенции |
   | `docs/roadmap.md` | roadmap |
   | `docs/backlog/NNN-feature-name/` | фичи и планы (БЕЗ подпапок active/archive) |
   | `docs/research/` | результаты researcher |
   | `docs/agent-learnings/<agent>/YYYY-MM-DD_slug.md` | learnings |

   Запрещены X0-пути: `docs/overview.md`, `docs/planning/vision.md`, `docs/backlog/current/`, `docs/backlog/active/`, `docs/conventions.md` (одним файлом), `/project/results/`, `session-history/`.
4. **Универсальность стека:** никакого хардкода технологий. Команды-примеры давать вариантами по стеку («npm audit / pip-audit / …») с инструкцией определить стек по `CLAUDE.md` и файлам зависимостей.
5. **Git централизован в @devops:** каждый новый агент в секции «НЕ ДЕЛАЙ» содержит строку «НЕ выполняй `git commit` / `git push` — делегируй @devops».
6. **Ревьюеры не правят код:** у `code-reviewer` и `security-reviewer` НЕТ Edit в tools; Write — только для отчёта ревью.
7. **Каждый новый агент содержит секции** (по образцу database-engineer): `## Role`, `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)`, workflow-секции, `## НЕ ДЕЛАЙ`, `## Agent Learnings` (путь `docs/agent-learnings/<name>/YYYY-MM-DD_slug.md`, «если директория есть в проекте»), `## Взаимодействие с другими агентами` (таблица From/To).
8. **Верификация каждого файла** (шаг «Verify» каждой задачи) — все три команды, из корня репо:

   ```bash
   grep -nE '\{\{|MANIFESTS_ROOT|AGENTS_ROOT|DOCS_ROOT|BACKLOG_ROOT' <file>.md   # ожидание: пусто
   grep -niE 'manifest|x0-|x0 framework|\bx0\b' <file>.md                        # ожидание: пусто
   grep -oE 'docs/[a-zA-Z0-9/_.-]*' <file>.md | sort -u                          # ожидание: только пути из таблицы п.3
   ```
9. **Источники X0** лежат в `/Users/serg1kk/Local Documents /Projects/X0-Framework/.x0/agents/` (далее `$X0`). Источники только читаем, никогда не меняем.
10. Коммит — после каждой задачи, сообщения в стиле репо (см. `git log --oneline`), с trailer `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.

---

### Task 1: Новый агент `code-reviewer`

**Files:**
- Create: `code-reviewer.md`
- Read (источники): `$X0/prompts/core/code-reviewer.md`, `$X0/manifests/specialized/quality/code-reviewer.md`, `database-engineer.md` (эталон формата), `implementation-plan-reviewer.md` (стиль ревьюера)

**Interfaces:**
- Produces: файл `code-reviewer.md` — строка для README-таблицы в Task 11: `| code-reviewer | sonnet | — | Review кода после разработчика: SOLID/DRY/KISS, соответствие плану, error handling. Итерирует с разработчиком, код не правит |`

- [ ] **Step 1: Прочитай оба источника и эталон формата целиком**

- [ ] **Step 2: Напиши `code-reviewer.md`**

Frontmatter (дословно):

```yaml
---
name: code-reviewer
description: Senior Code Reviewer для review качества кода после реализации. Вызывай после того как разработчик завершил задачу - проверка SOLID/DRY/KISS, соответствия implementation plan, error handling, maintainability. Находки возвращает разработчику и итерирует до чистого ревью. Код НЕ правит.
model: sonnet
color: orange
tools: Read, Grep, Glob, Bash, Write
---
```

Тело — merge источников в таком порядке (секции и что куда):

1. `## Role` — из промпта (Senior Software Engineer 10+ лет...), + добавить «Код НЕ правишь — находки возвращаются разработчику».
2. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — написать по образцу database-engineer: (1) `CLAUDE.md` — стек, конвенции; (2) implementation plan фичи в `docs/backlog/NNN-feature-name/plan.md` (если есть) — что должно быть реализовано; (3) `docs/conventions/` — стандарты кода; (4) `docs/ADR/` — архитектурные решения, которым код обязан соответствовать; (5) `git diff` / изменённые файлы — scope ревью (Bash: `git diff main...HEAD --stat` или диапазон, указанный в задаче).
3. `## Review Process` — из манифеста Steps 1–6 (Read Context → Review Code Structure → Check SOLID Principles → Security Scan → Performance Analysis → Provide Feedback), инлайн с примерами из манифеста. Security Scan оставить ЛЁГКИМ (hardcoded secrets, инъекции, missing authz — очевидное) и добавить строку: «Глубокий security-аудит — зона @security-reviewer, не дублируй его работу».
4. `## Architectural Principles Check` — из промпта: DRY, KISS, SOLID (расшифровка), YAGNI.
5. `## Review Categories` — из промпта: 🔴 Critical (blocks) / 🟡 Important / 🔵 Medium / ⚪ Low с примерами.
6. `## Чеклисты` — из промпта: Plan Compliance, Code Quality, Best Practices (три checkbox-блока).
7. `## Output Format` — из промпта (шаблон `# Code Review: [FEATURE]` с секциями Summary / Critical / Important / Notes / Positive Observations / Overall Score). Финальный статус как у implementation-plan-reviewer: `⚠️ NEEDS REVISION` (есть 🔴 или множественные 🟡) / `✅ APPROVED`.
8. `## Stage-Specific Focus` — из промпта: MVP (функциональность важнее совершенства, техдолг с документированием) vs Production (строгие стандарты, coverage, security).
9. `## Common Issues to Check` — из промпта: Performance (N+1, алгоритмы), Security (базовое), Maintainability (функции >50 строк, вложенность >3, magic numbers).
10. `## Mentoring Approach` — из промпта, сжать до 4–5 строк: объясняй «почему», отмечай хорошее, конкретные примеры.
11. `## НЕ ДЕЛАЙ` — НЕ правь код (нет Edit — только отчёт); НЕ выполняй `git commit`/`git push` — делегируй @devops; НЕ пропускай файлы из diff.
12. `## Agent Learnings` — стандартный абзац, путь `docs/agent-learnings/code-reviewer/YYYY-MM-DD_slug.md`.
13. `## Взаимодействие с другими агентами` — таблица: From @backend-developer/@frontend-developer/@database-engineer (код готов); To разработчику (находки, итерация до ✅); To @security-reviewer (после ✅ APPROVED передать фичу); эскалация: 3+ раунда без прогресса → пользователь.

DROP (не переносить): «Framework Documentation References», Escalation через tech architect (у нас его нет — эскалация на пользователя), Tools Integration «static analysis if available» → заменить на «линтеры проекта командами из CLAUDE.md».

- [ ] **Step 3: Verify** — три команды из Global Constraints п.8 на `code-reviewer.md`; плюс `head -8 code-reviewer.md` — frontmatter содержит все 5 полей, Edit отсутствует в tools.

- [ ] **Step 4: Commit**

```bash
git add code-reviewer.md
git commit -m "Add code-reviewer agent (merged from X0 core prompt + quality manifest)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: Новый агент `security-reviewer`

**Files:**
- Create: `security-reviewer.md`
- Read (источники): `$X0/prompts/specialized/security/security-auditor.md`, `$X0/manifests/specialized/security/security-auditor.md`, `database-engineer.md` (эталон)

**Interfaces:**
- Produces: `security-reviewer.md` — строка для README: `| security-reviewer | opus | — | Security-аудит кода после code review: OWASP Top 10, auth/authz, скан зависимостей и секретов. Работает точечно (фича) и аудитом всего репо |`

- [ ] **Step 1: Прочитай оба источника целиком** (манифест 467 строк — прочитать весь, там Steps 1–6 и Quality Checklist)

- [ ] **Step 2: Напиши `security-reviewer.md`**

Frontmatter (дословно):

```yaml
---
name: security-reviewer
description: Senior Security Reviewer для security-аудита. Вызывай после code review фичи или для аудита всего репо - OWASP Top 10, auth/authz deep-dive, скан зависимостей и секретов, data protection. Находки возвращает разработчику с severity и remediation. Код НЕ правит.
model: opus
color: red
tools: Read, Grep, Glob, Bash, Write
---
```

Тело — merge:

1. `## Role` — из промпта (Senior Security Auditor...), + «Два режима вызова: (1) фича после code review, (2) полный аудит репо. Код НЕ правишь».
2. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — (1) `CLAUDE.md` — стек (определяет команды сканирования); (2) `docs/ADR/` — решения с тегами `security`, `auth` (какой auth-механизм принят); (3) `docs/conventions/` — security-паттерны проекта, если описаны; (4) файлы зависимостей — менеджер пакетов для скана; (5) scope: фича (diff) или весь репо.
3. `## Workflow` — из манифеста Steps 1–6 инлайн, с кодовыми примерами манифеста: Step 1 Threat Modeling; Step 2 Code Security Review (паттерны auth/authz/input validation из манифеста); Step 3 Dependency Scan; Step 4 Auth/Authz Review (JWT verify not decode, cookie flags httpOnly/secure/sameSite, ownership-проверки); Step 5 Data Protection Review (шифрование, PII, secrets management); Step 6 Penetration Testing (сценарии из манифеста: доступ без auth, манипуляция user ID, инъекции в поля ввода).
4. `## OWASP Top 10 (2021) Coverage` — из промпта: полный список A01–A10 с расшифровками.
5. `## Scanning Commands` — из промпта, РАСШИРИТЬ оговоркой по стеку: «команды по стеку проекта, определи из CLAUDE.md и файлов зависимостей»:

```bash
# Зависимости (по стеку): npm audit --audit-level=high / pip-audit / go vet + govulncheck / bundler-audit
# Секреты: gitleaks detect --source .   (если установлен; иначе grep-паттерны на ключи/токены)
# Контейнеры (если есть Dockerfile): trivy image <image>
```

6. `## Quality Checklist` — из манифеста (полный чек-лист аудита).
7. `## Output Format` — отчёт: `# Security Review: [scope]`, секции по severity Critical/High/Medium/Low, для каждой находки: Location (file:line), Vulnerability (какая категория OWASP), Impact, Remediation (конкретный фикс). Финальный статус: `⚠️ NEEDS FIXES` (есть Critical/High) / `✅ PASSED`.
8. `## Key Principles` — из промпта: no system is 100% secure, defense in depth, compliance ≠ security, document everything.
9. `## НЕ ДЕЛАЙ` — НЕ правь код; НЕ выполняй git-операции — @devops; НЕ запускай деструктивные проверки (реальные эксплойты, DoS) — только чтение, статический анализ и безопасные сканеры; НЕ выноси секреты в отчёт (маскируй значения).
10. `## Agent Learnings` — путь `docs/agent-learnings/security-reviewer/YYYY-MM-DD_slug.md`.
11. `## Взаимодействие` — From @code-reviewer (фича после ✅) или User (аудит репо); To разработчику (находки Critical/High — блокируют, итерация); To @qa-engineer (после ✅ PASSED); To @devops (находки об инфре/CI).

DROP: Compliance-фреймворки как отдельная секция (GDPR/PCI-DSS оставить одной строкой в Role — «учитывай регуляторику проекта, если заявлена в CLAUDE.md»), «Working with Other Agents» из X0 (заменён нашей таблицей), ссылка «Detailed Workflow → manifest».

- [ ] **Step 3: Verify** — три команды п.8 Global Constraints; frontmatter: model opus, Edit отсутствует.

- [ ] **Step 4: Commit**

```bash
git add security-reviewer.md
git commit -m "Add security-reviewer agent (merged from X0 security-auditor prompt + manifest)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: Новый агент `researcher`

**Files:**
- Create: `researcher.md`
- Read (источники): `$X0/prompts/core/researcher.md`, `$X0/manifests/core/researcher.md` (649 строк — прочитать целиком), `database-engineer.md` (эталон)

**Interfaces:**
- Produces: `researcher.md` — строка для README: `| researcher | sonnet | context7 | Технические исследования: сравнение технологий, best practices, валидация гипотез. Research report с comparison-матрицей и рекомендацией |`

- [ ] **Step 1: Прочитай источники целиком**

- [ ] **Step 2: Напиши `researcher.md`**

Frontmatter (дословно):

```yaml
---
name: researcher
description: Senior Research Analyst для технических исследований. Вызывай на этапе брейншторма/спеки или по запросу - сравнение технологий и библиотек, best practices, валидация гипотез. Выход - структурированный research report с comparison-матрицей, источниками и рекомендацией.
model: sonnet
color: blue
tools: Read, Grep, Glob, Write, WebSearch, WebFetch
mcpServers:
  - context7
---
```

Тело — merge:

1. `## Role` — из манифеста Mission (unbiased, well-sourced, actionable; объясняет trade-offs).
2. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — (1) `CLAUDE.md` — стек (исследование должно учитывать реальный стек); (2) `docs/roadmap.md` — стадия проекта (MVP vs Production меняет рекомендации); (3) `docs/ADR/` — уже принятые решения (не предлагать то, что противоречит принятому без явной пометки «предлагаю пересмотреть ADR-N»); (4) кодовая база (Grep/Glob) — что уже используется.
3. `## Pre-Research Checklist` — из манифеста: сформулировать ключевые вопросы, определить критерии сравнения ДО поиска, зафиксировать scope.
4. `## Methodology` — из промпта + манифеста: Initial Analysis → Data Collection (официальная документация приоритетнее блогов; проверка дат актуальности; минимум 2–3 независимых источника на вывод) → Synthesis (паттерны, trade-offs, рекомендация).
5. `## Comparison Matrix` — из манифеста: шаблон таблицы сравнения (criteria × options, weights, вердикт).
6. `## Output Format` — из промпта: структура Research Report (Executive Summary 3–5 пунктов → Detailed Analysis → Recommendations: immediate/long-term/risks → Sources). Сохранение: `docs/research/YYYY-MM-DD-topic.md` (если папка `docs/research/` есть в проекте; иначе — отдать отчёт в ответе).
7. `## Quality Standards` — из промпта: данные актуальны, источники достоверны, выводы обоснованы, рекомендации actionable + из манифеста: каждый вывод привязан к источнику.
8. `## context7 MCP (если подключён)` — актуальная документация библиотек при сравнении API/возможностей; использовать вместо устаревших знаний.
9. `## НЕ ДЕЛАЙ` — НЕ выполняй git-операции — @devops; НЕ давай рекомендации без источников; НЕ смешивай факты и мнения (помечай мнения).
10. `## Agent Learnings` — путь `docs/agent-learnings/researcher/YYYY-MM-DD_slug.md`.
11. `## Взаимодействие` — From User / @implementation-plan-architect (вопрос на этапе брейншторма или планирования); To вызвавшему (report); результаты исследований интегрируются в план (@implementation-plan-reviewer проверяет гейтом «Результаты исследований интегрированы»).

DROP: «Framework Documentation References», `{{DOCS_ROOT}}explorations/`, `session-history/`, «Market Overview / Competitors Analysis» из Output (бизнес-исследования рынка — не наш кейс, оставить фокус на технических исследованиях; competitors упомянуть одной строкой «сравнение аналогов — по запросу»).

- [ ] **Step 3: Verify** — три команды п.8 Global Constraints на `researcher.md`.

- [ ] **Step 4: Commit**

```bash
git add researcher.md
git commit -m "Add researcher agent (merged from X0 core prompt + manifest)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 4: Новый агент `accessibility-expert`

**Files:**
- Create: `accessibility-expert.md`
- Read (источники): `$X0/manifests/specialized/design/accessibility-expert.md` (535 строк — целиком), `database-engineer.md` (эталон), `designer.md` и `frontend-developer.md` (границы ролей)

**Interfaces:**
- Produces: `accessibility-expert.md` — строка для README: `| accessibility-expert | sonnet | playwright | WCAG-аудит (axe-core, Lighthouse, Pa11y), ревью дизайна на a11y, самостоятельный фикс a11y-проблем в коде. По вызову |`

- [ ] **Step 1: Прочитай источники целиком**

- [ ] **Step 2: Напиши `accessibility-expert.md`**

Frontmatter (дословно):

```yaml
---
name: accessibility-expert
description: Accessibility Expert для WCAG-аудита и фиксов a11y. Вызывай точечно когда нужна проверка доступности - аудит (axe-core, Lighthouse, Pa11y), ревью дизайна на контраст/фокус/семантику, самостоятельный фикс a11y-проблем в коде.
model: sonnet
color: green
tools: Read, Grep, Glob, Bash, Edit, Write
mcpServers:
  - playwright
---
```

Тело — merge (источник один, манифест):

1. `## Role` — из Mission манифеста + «вызывается точечно, не встроен в обязательный пайплайн».
2. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — (1) `CLAUDE.md` — стек, как запустить приложение; (2) `DESIGN.md` (корень) или `docs/design-system.md` — токены контраста, focus-стили, семантика компонентов; (3) фреймворк FE по зависимостям (React/Vue/Svelte — меняет паттерны aria); (4) существующие тесты a11y (Grep: `axe`, `a11y`, `aria-`).
3. `## Workflow` — из манифеста Steps 1–5 инлайн: Audit (axe-core, Lighthouse accessibility, Pa11y — команды с оговоркой «если установлены; иначе ручной аудит по чек-листу WCAG») → Design Review (контраст ≥ 4.5:1 текст / 3:1 крупный, focus indicators, target size) → Code Implementation (семантический HTML прежде aria; aria — только когда семантики не хватает; keyboard navigation; фиксишь сам через Edit) → Testing & Validation (повторный скан + ручная клавиатурная проверка + Playwright MCP: accessibility snapshot страницы) → Documentation & Reporting.
4. `## Отчёт об аудите` — из манифеста: шаблон `# Accessibility Audit: [scope]` с Executive Summary, Critical Issues (block release: missing form labels, insufficient contrast, keyboard trap — примеры из манифеста), Serious/Moderate/Minor, для каждой: WCAG-критерий, Location, Fix.
5. `## Playwright MCP (если подключён)` — accessibility snapshot как основной инструмент проверки дерева доступности; проход user flow только с клавиатуры.
6. `## НЕ ДЕЛАЙ` — НЕ выполняй git-операции — @devops; НЕ меняй визуальный дизайн без согласования с @designer (контраст-фиксы — через токены дизайн-системы, если она есть); НЕ вешай aria на всё подряд (семантика первична).
7. `## Agent Learnings` — путь `docs/agent-learnings/accessibility-expert/YYYY-MM-DD_slug.md`.
8. `## Взаимодействие` — From User (точечный вызов) / @designer (ревью макета) / @qa-engineer (a11y-баги); To @frontend-developer (находки, которые требуют рефакторинга компонентов); To @designer (проблемы уровня дизайн-системы).

DROP: Required Reading с X0-путями, ссылки на конфиги.

- [ ] **Step 3: Verify** — три команды п.8 Global Constraints на `accessibility-expert.md`.

- [ ] **Step 4: Commit**

```bash
git add accessibility-expert.md
git commit -m "Add accessibility-expert agent (merged from X0 design manifest)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 5: Новый агент `performance-engineer`

**Files:**
- Create: `performance-engineer.md`
- Read (источники): `$X0/manifests/specialized/quality/performance-engineer.md` (469 строк — целиком), `database-engineer.md` (эталон + границы ролей: query-оптимизация есть у обоих)

**Interfaces:**
- Produces: `performance-engineer.md` — строка для README: `| performance-engineer | sonnet | playwright, postgres | Оптимизация производительности: baseline-замеры, поиск bottleneck-ов (N+1, bundle size), оптимизация, валидация против baseline. По вызову |`

- [ ] **Step 1: Прочитай источники целиком**

- [ ] **Step 2: Напиши `performance-engineer.md`**

Frontmatter (дословно):

```yaml
---
name: performance-engineer
description: Performance Engineer для оптимизации производительности. Вызывай точечно при перф-проблемах - baseline-замеры (Lighthouse, тайминги API и запросов), поиск bottleneck-ов (N+1, bundle size, медленные запросы), оптимизация и валидация против baseline.
model: sonnet
color: purple
tools: Read, Grep, Glob, Bash, Edit, Write
mcpServers:
  - playwright
  - postgres
---
```

Тело — merge:

1. `## Role` — из Mission + «Железное правило: сначала измерь, потом оптимизируй. Без baseline оптимизация не начинается».
2. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — (1) `CLAUDE.md` — стек, как запустить, где перф-таргеты если заданы; (2) `docs/ADR/` — теги `performance`, `architecture`; (3) профилировщики стека по зависимостям (Lighthouse для web, py-spy/pprof/clinic — по языку); (4) симптом от вызвавшего: ЧТО тормозит (страница, endpoint, запрос, сборка).
3. `## Workflow` — из манифеста Steps 1–4 инлайн: Baseline Measurement (Lighthouse, тайминги API, `EXPLAIN ANALYZE` ключевых запросов — числа фиксируются в отчёте) → Bottleneck Identification (что занимает больше всего: DB / processing / external APIs; проверка N+1, bundle size, blocking operations) → Optimization Implementation (фиксишь сам через Edit; по одному изменению за раз, чтобы видеть вклад каждого) → Validation (повторные замеры против baseline, числа до/после в отчёте).
4. `## Частые bottleneck-и` — из манифеста: N+1 queries, missing indexes, `SELECT *`, отсутствие пагинации, большой bundle (code splitting, lazy loading), блокирующие операции в горячем пути, отсутствие кэширования повторяемых вычислений.
5. `## MCP (если подключены)` — postgres (read-only): `EXPLAIN ANALYZE`, инспекция индексов; playwright: замер реального времени загрузки страниц, повтор user flow до/после.
6. `## Границы с @database-engineer` — одиночный медленный запрос/индекс — может решить сам; редизайн схемы, миграции — передать @database-engineer.
7. `## Output Format` — отчёт: baseline → находки (по вкладу в проблему) → применённые фиксы → числа до/после → что осталось (backlog-рекомендации).
8. `## НЕ ДЕЛАЙ` — НЕ выполняй git-операции — @devops; НЕ оптимизируй без замеров (premature optimization); НЕ меняй поведение/API ради перфа без согласования; НЕ применяй миграции БД — @database-engineer/@devops.
9. `## Agent Learnings` — путь `docs/agent-learnings/performance-engineer/YYYY-MM-DD_slug.md`.
10. `## Взаимодействие` — From User / @qa-engineer (перф-баг) / @error-detective (root cause = перф); To @database-engineer (схема/миграции); To @devops (инфраструктурные фиксы: кэш-слой, CDN).

DROP: load testing со 100 concurrent users как обязательный шаг (оставить одной строкой «load testing — по запросу, инструментом проекта»), X0-пути.

- [ ] **Step 3: Verify** — три команды п.8 Global Constraints на `performance-engineer.md`.

- [ ] **Step 4: Commit**

```bash
git add performance-engineer.md
git commit -m "Add performance-engineer agent (merged from X0 quality manifest)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 6: Новый агент `error-detective`

**Files:**
- Create: `error-detective.md`
- Read (источники): `$X0/manifests/specialized/debugging/error-detective.md` (123 строки), `database-engineer.md` (эталон)

**Interfaces:**
- Produces: `error-detective.md` — строка для README: `| error-detective | sonnet | — | Расследование инцидентов: анализ логов, корреляция ошибок, root cause. Обязан использовать скилл superpowers:systematic-debugging, если доступен. По вызову |`

- [ ] **Step 1: Прочитай источники целиком**

- [ ] **Step 2: Напиши `error-detective.md`**

Frontmatter (дословно):

```yaml
---
name: error-detective
description: Error Detective для расследования инцидентов и ошибок. Вызывай при дебаге - анализ логов, корреляция ошибок во времени и между компонентами, поиск root cause. Обязан использовать скилл superpowers:systematic-debugging, если тот доступен.
model: sonnet
color: cyan
tools: Read, Grep, Glob, Bash, Skill
---
```

Тело:

1. `## Role` — из Overview манифеста: расследование ошибок по логам, стек-трейсам и симптомам; выход — root cause + рекомендованный фикс (сам не чинит — передаёт разработчику).
2. `## Обязательный скилл` — дословно вставить:

```markdown
## Обязательный скилл

Первым шагом проверь, доступен ли скилл `superpowers:systematic-debugging` (есть в списке доступных скиллов). Если доступен — ОБЯЗАТЕЛЬНО вызови его и веди всё расследование по его процессу (воспроизведение, гипотезы, сужение причины, верификация). Свою экспертизу по логам и корреляции применяй ВНУТРИ этого процесса. Если скилл недоступен — работай по Workflow ниже.
```

3. `## Определи контекст проекта (первый шаг, ОБЯЗАТЕЛЬНО)` — (1) `CLAUDE.md` — стек, где логи, как запустить; (2) симптом от вызвавшего: ошибка/стек-трейс/шаги воспроизведения; (3) `git log` недавних изменений (Bash) — что менялось перед появлением ошибки; (4) `docs/agent-learnings/` — не встречалась ли похожая проблема раньше.
4. `## Workflow` — из манифеста Process, инлайн: собрать все проявления ошибки (Grep по логам: сообщение, error code, timestamps) → корреляция (по времени: что происходило одновременно; по компонентам: где первая ошибка в цепочке, отличить причину от следствия) → гипотезы с ранжированием по правдоподобию → проверка гипотез (чтение кода, воспроизведение через Bash) → root cause + evidence.
5. `## Output Format` — отчёт: Симптом → Timeline (последовательность событий) → Root Cause (с доказательствами: конкретные строки логов/кода) → Рекомендованный фикс → Как предотвратить повторение (мониторинг/тест).
6. `## НЕ ДЕЛАЙ` — НЕ чини код сам (нет Edit) — root cause передаётся разработчику; НЕ выполняй git-операции — @devops; НЕ останавливайся на первом «подозреваемом» без evidence (корреляция ≠ причина).
7. `## Agent Learnings` — путь `docs/agent-learnings/error-detective/YYYY-MM-DD_slug.md`.
8. `## Взаимодействие` — From User / @qa-engineer (баг без понятной причины) / @devops (инцидент на проде); To @backend-developer/@frontend-developer/@database-engineer (root cause + фикс); To @performance-engineer (если причина — перф).

DROP: Dependencies/MCP Servers/Success Criteria в X0-формате (переработаны в секции выше), X0-пути.

- [ ] **Step 3: Verify** — три команды п.8 Global Constraints на `error-detective.md`. Дополнительно: `grep -n "systematic-debugging" error-detective.md` — минимум 2 вхождения (description + секция).

- [ ] **Step 4: Commit**

```bash
git add error-detective.md
git commit -m "Add error-detective agent (merged from X0 debugging manifest, wired to systematic-debugging skill)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 7: Обогащение `implementation-plan-architect`

**Files:**
- Modify: `implementation-plan-architect.md`
- Read (источники): `$X0/prompts/core/technical-architect.md`, `$X0/manifests/core/technical-architect.md` (878 строк — прочитать, брать выборочно), `$X0/manifests/specialized/backend/backend-architect.md`, `$X0/manifests/specialized/api/api-designer.md`, `$X0/prompts/specialized/architecture/backend-architect.md`, `$X0/prompts/specialized/architecture/api-designer.md`

**Interfaces:**
- Consumes: текущий `implementation-plan-architect.md` (структура: Role → Обязательно прочитай → Skills → Core Responsibilities → Critical Constraints → Key Working Rules → Pre-Planning Checklist → Plan Output Location → Stage-Specific Guidelines → Agent Coordination → Agent Learnings)
- Produces: тот же файл + новая секция `## Architectural Expertise`

- [ ] **Step 1: Прочитай текущий файл и источники**

- [ ] **Step 2: Вставь новую секцию `## Architectural Expertise (применяй при планировании)`** — после секции `## Key Working Rules`, перед `## Pre-Planning Checklist`. Содержимое (дословно, дополни деталями из источников только если они конкретнее):

```markdown
## Architectural Expertise (применяй при планировании)

### ADR-мышление
- Если решение затрагивает несколько компонентов, вводит технологию или меняет паттерн данных → пометь в плане «создать ADR» с кратким rationale (сам ADR пишется при реализации)
- Перед планированием прочитай существующие `docs/ADR/` — план НЕ должен им противоречить; если противоречие осознанно — явно пометь «предлагаю пересмотреть ADR-N»
- Anti-patterns, которых НЕ должно быть в плане: God Object (один модуль делает всё), циклические зависимости, premature optimization, over-engineering под несуществующую нагрузку

### Границы модулей и data flow
- Для фичи определи: какие модули затронуты, где проходят границы ответственности, как течёт data flow (простая диаграмма `A → B → C` в секции «Взаимодействие компонентов»)
- Выбор технологии/подхода — всегда с trade-offs: 2–3 варианта, рекомендация, почему (одним абзацем, без эссе)
- Complexity соответствует stage: MVP — монолит и одна БД это норма; выделение сервисов — только при доказанной необходимости

### API-контракты
Секция «API и интерфейсы» плана для КАЖДОГО нового endpoint определяет:
- Метод + путь (REST) или query/mutation (GraphQL) — naming по конвенциям проекта (проверь существующие endpoints через Grep)
- Вход/выход словами (без типов), критичные ошибки и коды ответов
- Auth: кто имеет доступ, какие проверки (ownership для update/delete)
- Пагинация/фильтрация для списочных endpoints; версионирование — только если в проекте уже принято
```

- [ ] **Step 3: Обнови `## Pre-Planning Checklist`** — добавь два пункта в конец списка:

```markdown
- [ ] Для каждого нового endpoint определены auth-требования и ошибки
- [ ] Решения с trade-offs: выбраны из 2-3 вариантов, помечены кандидаты в ADR
```

- [ ] **Step 4: Упрости пути бэклога** (решение пользователя: без подпапки `active/`). В `implementation-plan-architect.md` две замены:
  - строка `- **\`docs/backlog/active/\`** — текущие фичи в работе` → `- **\`docs/backlog/\`** — фичи в работе`
  - в `## Plan Output Location`: `docs/backlog/active/NNN-feature-name/plan.md` → `docs/backlog/NNN-feature-name/plan.md`

  Затем проверь остальные файлы репо: `grep -rn "backlog/active" *.md` — ожидание: пусто.

- [ ] **Step 5: Verify** — три команды п.8 Global Constraints на файле; структура: `grep -n "^## " implementation-plan-architect.md` — секция Architectural Expertise стоит между Key Working Rules и Pre-Planning Checklist; `grep -n "backlog/active" implementation-plan-architect.md` — пусто.

- [ ] **Step 6: Commit**

```bash
git add implementation-plan-architect.md
git commit -m "Enrich implementation-plan-architect: ADR thinking, module boundaries, API contracts (from X0 technical/backend/api architects)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 8: Обогащение `implementation-plan-reviewer`

**Files:**
- Modify: `implementation-plan-reviewer.md`
- Read (источники): `$X0/manifests/specialized/security/security-auditor.md` (Step 1: Threat Modeling — основа security-секции), текущий `implementation-plan-reviewer.md`

**Interfaces:**
- Consumes: текущая структура (Role → Обязательно прочитай → Core Responsibilities → Review Quality Gates → Issue Priority Levels → Stage-Specific Focus → Red Flags → Feedback Style → Output Format → Agent Coordination → Agent Learnings)
- Produces: тот же файл + секции `## Security Review плана` и `## Architecture Review плана`

- [ ] **Step 1: Прочитай текущий файл и Threat Modeling из манифеста security-auditor**

- [ ] **Step 2: Вставь две секции** — после `## Review Quality Gates`, перед `## Issue Priority Levels`. Содержимое (дословно):

```markdown
## Security Review плана (ОБЯЗАТЕЛЬНО, все stages)

Threat modeling lite: дыры на этапе плана стоят дёшево, после имплементации — дорого. Проверь план:

- [ ] **Данные:** какие новые данные появляются? Есть ли PII/чувствительные данные? Прописано, где хранятся и кто имеет доступ?
- [ ] **Auth/Authz:** для каждого нового endpoint/страницы указано, кто имеет доступ? Для update/delete — ownership-проверки?
- [ ] **Input validation:** задачи с пользовательским вводом содержат требование server-side валидации?
- [ ] **Секреты и конфиг:** новые ключи/токены — через env/secret manager, не в коде?
- [ ] **Зависимости:** новые библиотеки — живые, поддерживаемые? Нет заброшенных/малоизвестных без обоснования?
- [ ] **Attack surface:** новые точки входа (endpoints, webhooks, file upload) — для каждой rate limiting / ограничения размера и типа, где релевантно?

Находки формулируй как НОВЫЕ ЗАДАЧИ для плана (пример: «добавить задачу: rate limiting на /api/upload»), с приоритетом 🔴/🟡.

## Architecture Review плана

- [ ] Решения плана не противоречат существующим ADR (`docs/ADR/`, если есть)
- [ ] Новые архитектурные решения помечены «создать ADR»
- [ ] API-контракты: для новых endpoints определены метод, путь, вход/выход, ошибки, auth
- [ ] Границы модулей не размываются, зависимости направлены правильно, нет циклических
- [ ] План не создаёт то, что уже есть в кодовой базе (переиспользование проверено)
```

- [ ] **Step 3: Обнови `## Review Quality Gates`** — в блок MUST PASS (All Stages) добавь две строки в конец:

```markdown
- [ ] Security Review плана пройден (секция ниже)
- [ ] Architecture Review плана пройден (секция ниже)
```

Из блока MUST PASS (Production Only) УДАЛИ строку `- [ ] Security соображения учтены` (заменена обязательной секцией для всех stages).

- [ ] **Step 4: Обнови `## Core Responsibilities`** — добавь пункт `7. **Security & Architecture Review** - threat modeling lite и проверка архитектурной части плана (секции ниже)`.

- [ ] **Step 5: Verify** — три команды п.8 Global Constraints; `grep -n "^## " implementation-plan-reviewer.md` — новые секции между Review Quality Gates и Issue Priority Levels; `grep -c "Security" implementation-plan-reviewer.md` ≥ 4.

- [ ] **Step 6: Commit**

```bash
git add implementation-plan-reviewer.md
git commit -m "Enrich implementation-plan-reviewer: plan-stage security review (threat modeling lite) + architecture review

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 9: Обогащение `qa-engineer` (влить test-automator)

**Files:**
- Modify: `qa-engineer.md`
- Read (источники): `$X0/manifests/specialized/testing/test-automator.md`, `$X0/prompts/specialized/testing/test-automator.md` (если есть), текущий `qa-engineer.md`

**Interfaces:**
- Consumes: текущая структура (Role → Определи контекст → Core Responsibilities → Playwright MCP → Postgres MCP → Testing Strategy → Bug Severity → Implementation Process → Bug Report Format → НЕ ДЕЛАЙ → Agent Learnings → Взаимодействие)
- Produces: тот же файл + секция `## Режим работы` сразу после `## Role`

- [ ] **Step 1: Прочитай текущий файл и источники test-automator**

- [ ] **Step 2: Вставь секцию `## Режим работы (задаётся в постановке задачи)`** — сразу после `## Role`, перед `## Определи контекст проекта`. Содержимое (дословно):

```markdown
## Режим работы (задаётся в постановке задачи)

Тебе ставят задачу в ОДНОМ из двух режимов. Если режим не указан — СПРОСИ, не выбирай сам.

### Режим 1: Ручное тестирование
- Exploratory testing по implementation plan: happy path, edge cases, error cases
- Playwright MCP для прохода user flows (если подключён)
- Выход: bug reports + чек-лист пройденных сценариев. Автотесты НЕ пишутся.

### Режим 2: Автотесты
- Пирамида тестов (70% unit / 20% integration / 10% E2E) — см. Testing Strategy
- CI-интеграция: тесты запускаются командами проекта (из CLAUDE.md); критерий — зелёные локально И в CI
- Flaky-менеджмент: нестабильный тест → чини ПРИЧИНУ (явные waits, изоляция тестовых данных, мок времени/сети), НЕ ставь blind retry; не удалось починить → skip с комментарием-причиной + bug report
- Выход: тесты в репо по конвенциям проекта + отчёт о прогоне
```

- [ ] **Step 3: Добавь из test-automator в `## Testing Strategy`** (если в манифесте есть конкретика сверх текущей пирамиды — селекторы устойчивости E2E, изоляция данных): добавь подсекцией `### Автотесты: правила устойчивости` 3–5 пунктов. Если конкретики нет — пропусти шаг (текущая пирамида уже покрывает).

- [ ] **Step 4: Обнови `## Core Responsibilities`** пункт 2: `**Test Implementation** — unit/integration/E2E тесты фреймворками проекта (режим «Автотесты»)`.

- [ ] **Step 5: Verify** — три команды п.8 Global Constraints; `grep -n "Режим работы" qa-engineer.md` — секция сразу после Role.

- [ ] **Step 6: Commit**

```bash
git add qa-engineer.md
git commit -m "Enrich qa-engineer: explicit manual/autotest modes, CI integration, flaky management (from X0 test-automator)

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 10: Ревизия `designer` (условное обогащение)

**Files:**
- Modify (условно): `designer.md`
- Read (источники): `$X0/manifests/specialized/design/ux-engineer.md` (1373 строки — просмотреть по секциям), `$X0/manifests/specialized/design/ux-designer.md` (402), текущий `designer.md`

**Interfaces:**
- Consumes: текущий `designer.md`
- Produces: либо обогащённый `designer.md`, либо вывод «изменения не нужны» (зафиксировать в результате задачи)

- [ ] **Step 1: Прочитай `designer.md` целиком и оба X0-манифеста**

- [ ] **Step 2: Сравни по чек-листу.** Кандидаты на добавление — ТОЛЬКО если в `designer.md` этого нет И это применимо к универсальному агенту:
  - Wireframe-процесс: текстовые/ASCII wireframes с аннотациями состояний (empty/loading/error/success) до реализации UI
  - User flow документация: точки входа, шаги, состояния, точки выхода
  - Чек-лист interaction states для каждого интерактивного элемента (default/hover/focus/active/disabled/loading)
  - Правило «сначала структура, потом стилизация»

  НЕ добавлять: X0-шаблоны с путями, дублирование того, что уже есть (у designer уже есть дизайн-система/DESIGN.md-workflow, токены, типографика).

- [ ] **Step 3: Если нашлись 1+ кандидата** — добавь секцию `## UX Process` с ними (сжато, по образцу существующих секций файла) и выполни Verify + Commit. **Если кандидатов нет** — задача завершена без изменений, коммит НЕ нужен; в отчёте по задаче напиши «designer уже покрывает X0 ux-манифесты, изменения не нужны».

- [ ] **Step 4: Verify (только если были изменения)** — три команды п.8 Global Constraints на `designer.md`.

- [ ] **Step 5: Commit (только если были изменения)**

```bash
git add designer.md
git commit -m "Enrich designer: UX process (wireframes, user flows, interaction states) from X0 ux manifests

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 11: README + финальная верификация репо

**Files:**
- Modify: `README.md`
- Test: финальный grep-sweep всех `*.md` в корне

**Interfaces:**
- Consumes: строки README-таблицы из Interfaces задач 1–6; факт изменений из задач 7–10

- [ ] **Step 1: Обнови таблицу агентов в README** — добавь 6 строк (значения из Interfaces задач 1–6) в логичном порядке: ревьюеры после implementation-plan-reviewer, точечные специалисты в конце. Проверь, что описания существующих строк не устарели (qa-engineer: добавь упоминание двух режимов).

- [ ] **Step 2: Замени секцию `## Workflow`** на (дословно):

````markdown
## Workflow

```
Брейншторм (режим РОЛИ, интерактивно с пользователем):
  User + @implementation-plan-architect (+ @designer, + @researcher по нужде)
  → спека → план → @implementation-plan-reviewer (полнота + архитектура + security плана) до ✅ APPROVED

Имплементация (сабагенты, автоматически):
  @backend-developer / @frontend-developer / @database-engineer / @designer (по плану, параллельно)
  → @code-reviewer (качество) ⇄ разработчик (до ✅)
  → @security-reviewer (безопасность) ⇄ разработчик (до ✅)
  → @qa-engineer (ручное или автотесты — режим задаётся в постановке)
  → @devops (commit / push / deploy / monitoring)

По вызову (не в обязательном пайплайне):
  @accessibility-expert · @performance-engineer · @error-detective · @researcher
```
````

Сохрани существующий абзац про `@database-engineer` и правило «git-операции централизованы в @devops».

- [ ] **Step 3: Финальный sweep всего репо** (из корня):

```bash
grep -nE '\{\{|MANIFESTS_ROOT|AGENTS_ROOT|DOCS_ROOT|BACKLOG_ROOT' *.md          # пусто
grep -rn "backlog/active\|backlog/current" *.md                                 # пусто (путь упрощён до docs/backlog/)
grep -niE 'манифест|manifest' *.md | grep -v README.md                          # пусто (README Credits — единственное легитимное место)
grep -hoE 'docs/[a-zA-Z0-9/_.-]*' *.md | sort -u                                # только канонические пути (таблица Global Constraints п.3)
grep -c '^| `' README.md                                                        # 14 строк-агентов в таблице
for f in code-reviewer security-reviewer; do grep -H "^tools:" $f.md | grep -v Edit || echo "FAIL: $f has Edit"; done
```

Ожидание: первые две команды пусто, пути канонические, 14 агентов, ревьюеры без Edit. Любое расхождение — исправить до коммита.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "Update README: 14 agents, workflow with review gates and on-demand specialists

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```
