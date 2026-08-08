# Claude Code Agents

Универсальные субагенты для [Claude Code](https://claude.com/claude-code) — работают в любом проекте независимо от стека. Каждый агент при запуске сам определяет контекст проекта (CLAUDE.md, файлы зависимостей, конвенции) и адаптируется под него.

Агенты и манифесты объединены в один файл на агента (frontmatter + инструкции), формат — стандартный для `.claude/agents/`.

## Агенты

| Agent | Model | MCP (опционально) | Роль |
|-------|-------|-------------------|------|
| `implementation-plan-architect` | opus | — | Создание детальных планов реализации: декомпозиция на задачи 1–4 часа, acceptance criteria, зависимости и параллельность, ADR-мышление, API-контракты |
| `implementation-plan-reviewer` | opus | — | Review планов: полнота, гранулярность, техническая корректность + security review плана (threat modeling lite) и architecture review. Итерирует с архитектором до approval |
| `code-reviewer` | sonnet | — | Review кода после разработчика: SOLID/DRY/KISS, соответствие плану, error handling. Итерирует с разработчиком, код не правит |
| `security-reviewer` | opus | — | Security-аудит кода после code review: OWASP Top 10, auth/authz, скан зависимостей и секретов. Работает точечно (фича) и аудитом всего репо |
| `backend-developer` | sonnet | postgres, context7 | Backend-задачи: API, модели данных, миграции, бизнес-логика |
| `database-engineer` | sonnet | postgres, context7 | Схемы БД, индексы, оптимизация запросов, безопасные миграции с rollback, data integrity |
| `frontend-developer` | sonnet | context7, playwright | Frontend-задачи: UI-фичи, компоненты, маршруты, интеграция с API + весь визуальный слой (токены → layout → компоненты → анимации) по дизайн-системе |
| `designer` | opus | playwright | Проектирование интерфейсов ДО кода: брейншторм экранов, user flows, ASCII-вайрфреймы, все состояния, UX-план и хендофф. Интерактивная роль (диалог), код не пишет |
| `qa-engineer` | sonnet | playwright, postgres | Тестирование в двух режимах — ручное или автотесты (режим задаётся в постановке): тест-планы, bug reports, quality gate sign-off |
| `devops` | haiku | — | Git-операции, CI/CD, деплой, инфраструктура, мониторинг |
| `researcher` | sonnet | context7 | Технические исследования: сравнение технологий, best practices, валидация гипотез. Research report с comparison-матрицей и рекомендацией |
| `accessibility-expert` | sonnet | playwright | WCAG-аудит (axe-core, Lighthouse, Pa11y), ревью дизайна на a11y, самостоятельный фикс a11y-проблем в коде. По вызову |
| `performance-engineer` | sonnet | playwright, postgres | Оптимизация производительности: baseline-замеры, поиск bottleneck-ов (N+1, bundle size), оптимизация, валидация против baseline. По вызову |
| `error-detective` | sonnet | — | Расследование инцидентов: анализ логов, корреляция ошибок, root cause. Использует скилл systematic-debugging, если доступен. По вызову |

MCP-серверы опциональны: агент использует их, если они подключены в проекте (`.mcp.json`) или глобально; их отсутствие не ломает агента.

## Workflow

```
Брейншторм (режим РОЛИ, интерактивно с пользователем):
  User + @designer (для UI-фич: проектирование экранов, флоу, вайрфреймы — до планирования)
  → @implementation-plan-architect (+ @researcher по нужде)
  → спека → план → @implementation-plan-reviewer (полнота + архитектура + security плана) до ✅ APPROVED

Имплементация (сабагенты, автоматически):
  @backend-developer / @frontend-developer / @database-engineer (по плану, параллельно)
  → @code-reviewer (качество) ⇄ разработчик (до ✅)
  → @security-reviewer (безопасность) ⇄ разработчик (до ✅)
  → @qa-engineer (ручное или автотесты — режим задаётся в постановке)
  → @devops (commit / push / deploy / monitoring)

По вызову (не в обязательном пайплайне):
  @accessibility-expert · @performance-engineer · @error-detective · @researcher
```

`@designer` — интерактивная роль: входить в диалог, не запускать fire-and-forget (вся ценность в вопросах, вариантах и гейтах). Механику брейншторма даёт скилл проекта (superpowers:brainstorming, grilling или другой) — в промпт агента она не зашита. Результат — UX-план, вайрфреймы и дизайн-спеки; реализует их @frontend-developer, которому передан и весь визуальный слой (токены, стили, анимации).

`@database-engineer` подключается точечно: схема/миграция для фичи (до или параллельно с @backend-developer), медленные запросы, data-level дебаг. Production-миграции применяет @devops по его rollback-плану.

Ключевое правило: **git-операции централизованы в @devops** — разработчики и QA не коммитят сами, а передают готовую работу DevOps-агенту.

## Установка

Скопируй нужных агентов в проект или глобально:

```bash
# В конкретный проект
cp *.md /path/to/project/.claude/agents/

# Глобально для всех проектов
cp *.md ~/.claude/agents/
```

`README.md` копировать не нужно.

## Принципы

- **Универсальность** — никакого хардкода стека: агент определяет технологии по `CLAUDE.md` и файлам зависимостей
- **Adapt to Existing** — новый код пишется в стиле существующего кода проекта
- **Опциональные конвенции** — ссылки на `docs/conventions/`, `docs/ADR/`, дизайн-систему (`DESIGN.md` в корне или `docs/design-system.md`) работают, если эти файлы есть; их отсутствие не ломает агента
- **Дизайн-система first** — designer и frontend-developer читают `DESIGN.md` перед любой UI-работой; конфликт кода и дизайн-системы решается в пользу дизайн-системы. Приоритет визуальных решений: слова пользователя → дизайн-система → скилл `artifact-design` → вкус агента
- **Agent Learnings** — агенты фиксируют ошибки и ограничения в `docs/agent-learnings/` (если директория есть в проекте)

## Credits

Роли и workflow-манифесты основаны на [X0-Framework](https://github.com/Serg1kk/X0-Framework) — meta-framework для Documentation-Driven Development с мультиагентной архитектурой.
