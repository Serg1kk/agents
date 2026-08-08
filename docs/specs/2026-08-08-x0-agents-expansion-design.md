# Дизайн: расширение базы агентов из X0-Framework

**Дата:** 2026-08-08
**Статус:** Approved (брейншторм с пользователем)
**Источник ролей:** `X0-Framework/.x0/agents/` (prompts + manifests)

## Цель

Расширить базу универсальных Claude Code агентов ролями из X0-Framework: добавить security-гейты в workflow (на этапе плана и после реализации), code review как обязательный этап, точечных специалистов по вызову. Каждый агент — один самодостаточный файл: промпт и манифест объединены, внешних ссылок на файлы X0 нет.

## Ключевые решения (из брейншторма)

1. **Security разделяем на два уровня:**
   - Security review **плана** — не отдельный агент, а новая секция в `implementation-plan-reviewer` (ловим дыры до имплементации, экономим токены).
   - Security review **кода** — отдельный агент `security-reviewer`, гейт после code review.
2. **Code-reviewer и security-reviewer — раздельные агенты.** Разные mindset'ы: «хороший ли код» vs «как это сломать». У code-reviewer остаётся лёгкая security-секция (очевидное: hardcoded secrets, инъекции), глубокий аудит — у security-reviewer.
3. **Архитекторов не плодим.** Экспертиза technical-architect, backend-architect и api-designer вливается в `implementation-plan-architect` (и проверка этой части — в `implementation-plan-reviewer`). Отдельные агенты не создаются: пользователь работает с IP-архитектором в режиме РОЛИ на этапе брейншторма, там и нужна вся плановая экспертиза.
4. **test-automator не отдельный** — вливается в `qa-engineer`. QA получает два явных режима: ручное тестирование / автотесты; режим указывается в постановке задачи, агент не выбирает сам.
5. **error-detective** — отдельный агент; в промпте обязательство: если доступен скилл `superpowers:systematic-debugging` — вызвать его и работать по нему.
6. **Ревьюеры не правят код** (принцип из X0 sandbox-профилей): у `code-reviewer` и `security-reviewer` нет Edit — только чтение, анализ и отчёт. Находки возвращаются разработчику.
7. **`*-security.md` файлы X0** (`manifests/specialized/security/*-security.md`) — это sandbox-профили оркестратора, НЕ методология. В контент агентов не переносятся; единственное наследие — read-only набор тулов у ревьюеров (п. 6).
8. **database-engineer уже в репо** (перенесён ранее из database-architect) — не трогаем.

## Новые агенты (6 файлов)

| Агент | Модель | Tools (frontmatter) | MCP (опц.) | Источники в X0 |
|---|---|---|---|---|
| `code-reviewer` | sonnet | Read, Grep, Glob, Bash, Write | — | `prompts/core/code-reviewer.md` + `manifests/specialized/quality/code-reviewer.md` |
| `security-reviewer` | opus | Read, Grep, Glob, Bash, Write | — | `prompts/specialized/security/security-auditor.md` + `manifests/specialized/security/security-auditor.md` |
| `researcher` | sonnet | Read, Grep, Glob, WebSearch, WebFetch, Write | context7 | `prompts/core/researcher.md` + `manifests/core/researcher.md` |
| `accessibility-expert` | sonnet | Read, Grep, Glob, Bash, Edit, Write | playwright | `manifests/specialized/design/accessibility-expert.md` |
| `performance-engineer` | sonnet | Read, Grep, Glob, Bash, Edit, Write | playwright, postgres | `manifests/specialized/quality/performance-engineer.md` |
| `error-detective` | sonnet | Read, Grep, Glob, Bash, Skill | — | `manifests/specialized/debugging/error-detective.md` |

### Роли и место в workflow

- **code-reviewer** — после разработчика, перед security-review: SOLID/DRY/KISS/YAGNI, соответствие implementation-плану, error handling, maintainability, базовый перформанс, лёгкая security-секция. Категории находок: Critical / Important / Medium / Low. Итерирует с разработчиком до чистого ревью.
- **security-reviewer** — после code-review: полный OWASP Top 10 (2021), auth/authz deep-dive (JWT, сессии, ownership-проверки), скан зависимостей (`npm audit` / `pip-audit`), секреты (`gitleaks`), data protection (шифрование, PII), pen-test сценарии из манифеста. Работает и точечно (фича), и аудитом всего репо.
- **researcher** — этап брейншторма/спеки и по запросу: сравнение технологий (comparison-матрицы), best practices, валидация гипотез. Выводы — actionable, с источниками и рекомендацией. Результаты — в `docs/research/` (если папка есть в проекте).
- **accessibility-expert** — по вызову: WCAG-аудит (axe-core, Lighthouse, Pa11y), ревью дизайна, самостоятельный фикс a11y-проблем в коде (есть Edit), отчёт с приоритетами.
- **performance-engineer** — по вызову: baseline-замеры → поиск bottleneck'ов (N+1, bundle size, медленные запросы) → оптимизация (есть Edit) → валидация против baseline.
- **error-detective** — по вызову при инцидентах: анализ логов, корреляция ошибок, root cause. Обязан вызвать `superpowers:systematic-debugging`, если скилл доступен.

## Обогащение существующих (3–4 файла)

- **`implementation-plan-architect`** — вливается плановая экспертиза:
  - из technical-architect: ADR-мышление (когда решение требует ADR), stage-awareness (MVP vs Production), архитектурные anti-patterns;
  - из backend-architect: границы сервисов/модулей, data flow, выбор технологий с trade-offs;
  - из api-designer: API-контракты (REST/GraphQL выбор, auth-флоу, пагинация, версионирование, error handling) — углубление существующей секции «API и интерфейсы».
  - Остаётся режимом РОЛИ (brainstorming + writing-plans), не сабагентом.
- **`implementation-plan-reviewer`** — две новые секции ревью:
  - **Security Review плана** (threat modeling lite): какие данные/PII появляются, прописаны ли требования auth/authz, input validation, обращение с секретами/конфигом, риски новых зависимостей. Материал — из Step 1 (Threat Modeling) манифеста security-auditor.
  - **Architecture Review плана**: соответствие существующим ADR, качество API-контрактов, границы модулей.
- **`qa-engineer`** — вливается test-automator: пирамида тестов, CI-интеграция автотестов, flaky-менеджмент. Два явных режима работы — **ручное тестирование / автотесты** — режим задаётся в постановке задачи.
- **`designer`** — сравнить с `manifests/specialized/design/ux-engineer.md` (1373 стр.) и `ux-designer.md` (402 стр.); добавить недостающее (wireframe-процесс, UX-паттерны) без раздувания. Если всё покрыто — не трогать.

## Принципы мерджа и чистоты (требования пользователя)

1. **Ноль внешних ссылок.** В итоговых промптах не остаётся: `{{MANIFESTS_ROOT}}`, `{{AGENTS_ROOT}}`, `{{DOCS_ROOT}}`, `{{BACKLOG_ROOT}}`, ссылок вида «см. манифест …», ссылок на X0-конфиги (`agents/configs/…`), tree metaphor, sandbox-специфики. Всё полезное из манифеста — инлайн в промпт.
2. **Канонические пути** — все агенты (новые и обогащаемые) ссылаются только на пути из таблицы ниже, всегда с оговоркой «если есть в проекте»:

   | Путь | Назначение |
   |---|---|
   | `CLAUDE.md` | стек, архитектура, deployment проекта |
   | `DESIGN.md` (корень) или `docs/design-system.md` | дизайн-система |
   | `docs/ADR/` | архитектурные решения |
   | `docs/conventions/` (`git.md`, `testing.md`, …) | конвенции |
   | `docs/roadmap.md` | roadmap |
   | `docs/backlog/active/NNN-feature-name/` | активные фичи и планы |
   | `docs/research/` | результаты researcher |
   | `docs/agent-learnings/<agent>/YYYY-MM-DD_slug.md` | learnings агентов |

   Никаких `docs/overview.md`, `docs/planning/vision.md`, `docs/backlog/current/…` и прочих X0-путей.
3. **Универсальность стека** — без хардкода технологий: агент определяет стек по `CLAUDE.md` и файлам зависимостей; примеры команд (npm audit / pip-audit и т.п.) даются как варианты по стеку.
4. **Формат файла** — как у существующих: frontmatter (`name`, `description` с триггером вызова, `model`, `color`, `tools`) + тело. Язык и стиль — как в текущих агентах репо (микс RU/EN, описания на русском).
5. **Git-операции централизованы в @devops** — новые агенты не коммитят (правило репо сохраняется).

## Обновлённый workflow (для README)

```
Брейншторм (режим РОЛИ):
  User + @implementation-plan-architect (+ @designer, + @researcher по нужде)
  → спека → план → @implementation-plan-reviewer (полнота + архитектура + security плана) до ✅ APPROVED

Имплементация (сабагенты, автоматически):
  @backend-developer / @frontend-developer / @database-engineer / @designer (по плану, параллельно)
  → @code-reviewer (качество) ⇄ разработчик (до чистого ревью)
  → @security-reviewer (безопасность) ⇄ разработчик
  → @qa-engineer (ручное или автотесты — по постановке)
  → @devops (commit / push / deploy / monitoring)

По вызову:
  @accessibility-expert, @performance-engineer, @error-detective, @researcher
```

Итого: 8 существующих + 6 новых = 14 агентов.

## Верификация (acceptance)

После имплементации по каждому файлу:

1. `grep -rn "{{" *.md` — пусто (нет плейсхолдеров).
2. `grep -rniE "manifest|x0|AGENTS_ROOT" *.md` — пусто (кроме легитимных упоминаний в README Credits).
3. `grep -hoE 'docs/[a-zA-Z0-9/_.-]*' *.md | sort -u` — только канонические пути из таблицы.
4. Frontmatter каждого нового агента валиден (name, description, model, tools).
5. README: таблица агентов и workflow-схема обновлены, database-engineer-абзац не сломан.
6. У `code-reviewer` и `security-reviewer` нет Edit в tools.

## Out of scope

- Отдельные агенты technical-architect, backend-architect, api-designer, test-automator, cloud-architect, monitoring-engineer, database-optimizer, git-commit-helper, code-refactorer, project-specific агенты (api-security, blockchain-*, ecommerce-*, saas-*).
- Оркестрация X0 (`orchestration/`, configs, sandbox) — не переносится.
- Динамическая подгрузка контекста (ссылки на внешние манифесты) — сознательно не используется.
