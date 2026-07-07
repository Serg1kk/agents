# Claude Code Agents

Универсальные субагенты для [Claude Code](https://claude.com/claude-code) — работают в любом проекте независимо от стека. Каждый агент при запуске сам определяет контекст проекта (CLAUDE.md, файлы зависимостей, конвенции) и адаптируется под него.

Агенты и манифесты объединены в один файл на агента (frontmatter + инструкции), формат — стандартный для `.claude/agents/`.

## Агенты

| Agent | Model | Роль |
|-------|-------|------|
| `implementation-plan-architect` | opus | Создание детальных планов реализации: декомпозиция на задачи 1–4 часа, acceptance criteria, зависимости и параллельность |
| `implementation-plan-reviewer` | opus | Review планов: полнота, гранулярность, техническая корректность. Итерирует с архитектором до approval |
| `backend-developer` | sonnet | Backend-задачи: API, модели данных, миграции, бизнес-логика |
| `frontend-developer` | sonnet | Frontend-задачи: UI-фичи, компоненты, маршруты, интеграция с API |
| `designer` | sonnet | Premium UX/UI: дизайн-система, токены, типографика, анимации |
| `qa-engineer` | sonnet | Тестирование: тест-планы, автотесты, bug reports, quality gate sign-off |
| `devops` | sonnet | Git-операции, CI/CD, деплой, инфраструктура, мониторинг |

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
- **Опциональные конвенции** — ссылки на `docs/conventions/`, `docs/ADR/`, `docs/design-system.md` работают, если эти файлы есть; их отсутствие не ломает агента
- **Agent Learnings** — агенты фиксируют ошибки и ограничения в `docs/agent-learnings/` (если директория есть в проекте)

## Credits

Роли и workflow-манифесты основаны на [X0-Framework](https://github.com/Serg1kk/X0-Framework) — meta-framework для Documentation-Driven Development с мультиагентной архитектурой.
