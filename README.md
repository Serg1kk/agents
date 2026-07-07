# Claude Code Agents

Универсальные субагенты для [Claude Code](https://claude.com/claude-code) — работают в любом проекте независимо от стека. Каждый агент при запуске сам определяет контекст проекта (CLAUDE.md, файлы зависимостей, конвенции) и адаптируется под него.

Агенты и манифесты объединены в один файл на агента (frontmatter + инструкции), формат — стандартный для `.claude/agents/`.

## Агенты

| Agent | Model | MCP (опционально) | Роль |
|-------|-------|-------------------|------|
| `implementation-plan-architect` | opus | — | Создание детальных планов реализации: декомпозиция на задачи 1–4 часа, acceptance criteria, зависимости и параллельность |
| `implementation-plan-reviewer` | opus | — | Review планов: полнота, гранулярность, техническая корректность. Итерирует с архитектором до approval |
| `backend-developer` | sonnet | postgres, context7 | Backend-задачи: API, модели данных, миграции, бизнес-логика |
| `frontend-developer` | sonnet | context7, playwright | Frontend-задачи: UI-фичи, компоненты, маршруты, интеграция с API |
| `designer` | sonnet | playwright | Premium UX/UI: дизайн-система, токены, типографика, анимации |
| `qa-engineer` | sonnet | playwright, postgres | Тестирование: тест-планы, автотесты, bug reports, quality gate sign-off |
| `devops` | haiku | — | Git-операции, CI/CD, деплой, инфраструктура, мониторинг |

MCP-серверы опциональны: агент использует их, если они подключены в проекте (`.mcp.json`) или глобально; их отсутствие не ломает агента.

## Workflow

```
User → @implementation-plan-architect ⇄ @implementation-plan-reviewer (до ✅ APPROVED)
     → @backend-developer / @frontend-developer / @designer (параллельно по плану)
     → @qa-engineer (quality gate)
     → @devops (commit / push / deploy / monitoring)
```

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
