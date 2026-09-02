---
name: vector-ai-team
description: Use when organizing an AI agent team for development — roles, workflow, harness, orchestrator design. Triggers on 'ИИ-команда', 'agent team', 'оркестратор агентов', 'harness'.
version: 1.0.0
author: Osmosy
license: MIT
metadata:
  hermes:
    tags: [agents, orchestration, harness, workflow, roles, development]
    related_skills: [delegation-gate-checklist, kanban-orchestrator, doubt-driven-review, hermes-agent]
---

# Vector AI Team — организация разработки командой ИИ-агентов

Методология сборки команды ИИ-агентов для разработки ПО: роли, workflow,
harness, границы ответственности. Основа — конспект «Базовая организация
разработки командой ИИ-агентов» (файл «Организация ИИ-команды.md»,
сентябрь 2026) + проверка фактов по первоисточникам (Symphony подтверждена
блогом OpenAI, апрель 2026).

## Главный принцип

**Главное — не промпты персонажей, а исполняемые ограничения.**
Независимость агентов достигается отдельными контекстами, полномочиями и
критериями результата — не разными моделями.

Четыре функции вместо десяти агентов:
1. планирование (Planning Agent);
2. разработка (Developer Agent);
3. независимая проверка (Reviewers);
4. интеграция и эксплуатация (Test/Integration + Incident).

Разделение ответственности:
```
Трекер    — ЧТО делать
Роль      — КАК действовать
Оркестратор — КОГО и КОГДА запускать
CI        — прошёл ли результат объективные проверки
Человек   — приоритеты, продуктовые компромиссы, security, prod
```

## Роль агента (обязательные поля)

- назначение; какие данные получает; источники истины;
- доступные инструменты; что может изменять; что запрещено;
- обязательные проверки; формат результата;
- критерии PASS / FAIL / BLOCKED.

Роль — постоянная должностная инструкция. Конкретная задача передаётся
отдельно (спецификация: цель, scope, acceptance criteria, ограничения,
ожидаемые тесты).

## Минимальные роли

| Роль | Делает | Ключевое ограничение |
|------|--------|---------------------|
| Planning Agent | цель, scope, AC, декомпозиция | не принимает продуктовых решений вне постановки |
| Developer Agent | код, тесты, PR | запрещено ослаблять тесты, менять AC, несвязанный рефакторинг |
| Correctness Reviewer | требования, границы, тесты | не изменяет код, новый контекст |
| Architecture Reviewer | границы модулей, зависимости, контракты | не изменяет код, новый контекст |
| Test & Integration | E2E, staging, логи, отчёт о дефектах | возвращает задачу или эскалирует |
| Incident Investigator | последовательность событий, локализация, issue | не чинит сам; передаёт разработчику |

Reviewer запускается в НОВОМ контексте и не имеет права изменять
проверяемый код. Ревьюеры запускаются параллельно; замечания
объединяются; цикл review → fix → review до прохождения или лимита.

## Workflow (12 шагов)

Трекер (Backlog → Spec → Ready → Dev → Agent Review → Integration →
Human Review → Deploy → Done) — основная панель управления.

1. Задача из трекера → 2. Planning создаёт спецификацию → 3. другой агент
проверяет спецификацию на полноту → 4. worktree/контейнер на задачу →
5. Developer реализует → 6. параллельные ревьюеры → 7. исправления →
8. цикл до прохождения или лимита → 9. CI (детерминированные проверки) →
10. staging → 11. интеграционные/E2E → 12. human approval → deploy →
мониторинг → новые задачи.

Definition of Done (минимум): AC выполнены; тесты добавлены; unit/
контрактные/интеграционные/архитектурные проверки проходят; в staging нет
новых ошибок; ревьюеры не нашли блокирующих; PR содержит проверяемые
результаты. ИИ не решает сам, что «достаточно хорошо».

## Harness

```
репозиторий + инструкции + инструменты + sandbox + worktree
+ workflow + тесты + CI + логи + права + retry + human approval
```

Хороший harness: воспроизводимая среда; ограниченные полномочия; короткий
цикл обратной связи; машинно проверяемый результат; сохранение состояния;
наблюдаемость; безопасное восстановление.

Критические архитектурные правила закрепляются КОДОМ в CI (запрещённые
зависимости, циклические импорты, схемы API, границы). Architecture
Reviewer интерпретирует сложные случаи; базовые инварианты проверяет CI.

## Harness-фреймворки (проверено 02.09.2026)

| Фреймворк | Что | Когда брать |
|-----------|-----|-------------|
| **OpenAI Symphony** (апрель 2026, Apache 2.0) | issue tracker как control plane; WORKFLOW.md (YAML front matter + промпт-шаблон); per-issue workspace; retry/reconcile; agent-agnostic | основа собственного минимального harness; +500% landed PR у команд OpenAI. Предупреждения: vague tickets → low-quality PRs; не production («low-key engineering preview»); не-Linear/не-Codex требует адаптеров |
| **OpenHands Software Agent SDK** | Python/TS/REST; 4 пакета (sdk/tools/workspace/agent_server); LocalWorkspace → DockerWorkspace → RemoteAPIWorkspace без смены кода агента | самая готовая открытая база исполнителей и reviewer-агентов; sandbox; production |
| **CrewAI** | Crews (автономные) + Flows (детерминированный порядок) | быстрое прототипирование ролевых команд; worktree/CI/PR интегрировать самому |
| **LangGraph** | граф состояний: typed state, conditional edges, durable checkpoints, interrupt/resume, HITL | свой строгий оркестратор; когда нужны пауза/резюме/audit trail |
| **AutoGen AgentChat** (Microsoft; в maintenance mode → agent-framework) | готовые типы команд: RoundRobinGroupChat, SelectorGroupChat, Swarm (handoff), граф | эксперименты с ролями; для dev-loop нужен свой harness |
| **mini-swe-agent** | ~100 строк Python, только bash, >74% SWE-bench verified; линейная история | компактный исполнитель внутри workflow; НЕ оркестратор |

## Практическая лестница

1. **Самый простой путь** (без оркестратора): GitHub Issues + 1 coding
   agent + worktree на задачу + 2 reviewer-агента + GitHub Actions +
   ручные статусы.
2. **Следующий уровень:** Linear/Issues + Symphony-подобный оркестратор +
   Codex/OpenHands исполнители + Actions + staging + observability.
3. **Своя платформа:** LangGraph/CrewAI Flows + OpenHands SDK + tracker API
   + worktree manager + CI/CD + своя permission policy.

Рациональная архитектура: Symphony-модель управления задачами + coding
agent-исполнитель + обычный GitHub CI как источник истины. НЕ начинать с
AutoGen/CrewAI-команды, свободно обсуждающей задачу: надёжнее фиксированный
конвейер с независимым ревью и детерминированными тестами.

## Связь с экосистемой Vector / Hermes

- `delegation-gate-checklist` — 4 шлюза перед spawn = «ограниченные
  полномочия» из этого документа.
- `kanban-orchestrator` + `kanban-worker` — реализация Symphony-модели
  на Hermes (task board как control plane).
- `doubt-driven-review` — независимый ревью в свежем контексте.
- `delegate_task` с enabled_toolsets = минимальный toolset роли.
- `vector-agent-ready` — смежная методология подготовки контента.

## Verification Checklist

- [ ] У каждой роли есть запреты и критерии PASS/FAIL/BLOCKED
- [ ] Reviewer не изменяет код и работает в новом контексте
- [ ] Базовые архитектурные инварианты проверяет CI, а не агент
- [ ] У каждой задачи есть AC и ожидаемые тесты ДО разработки
- [ ] Точки human approval определены (security, prod, миграции)
- [ ] Лимиты циклов review→fix заданы в workflow
- [ ] Логи/traces дают обратную связь после deploy