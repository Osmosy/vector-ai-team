<div align="center">

<img src="assets/vector-logo.png" alt="Vector AI Team" width="200"/>

# Vector AI Team

**Организация разработки командой ИИ-агентов — роли, workflow, harness, оркестратор**

[![Hermes Agent](https://img.shields.io/badge/Hermes-Agent-blue.svg)](https://github.com/NousResearch/hermes-agent)
[![Functions: 4](https://img.shields.io/badge/Functions-4-green.svg)](#минимальная-команда)
[![Roles: 6](https://img.shields.io/badge/Roles-6-orange.svg)](#минимальные-роли)
[![Frameworks: 6](https://img.shields.io/badge/Frameworks-6-blueviolet.svg)](#harness-фреймворки)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

Методология сборки команды ИИ-агентов для разработки ПО. Главный принцип:
**главное — не промпты персонажей, а исполняемые ограничения**. Независимость
агентов достигается отдельными контекстами, полномочиями и критериями
результата — не разными моделями и не промптами «персонажей».

Начинать не с виртуальной компании из десяти агентов, а с четырёх функций:
планирование, разработка, независимая проверка, интеграция и эксплуатация.

## Разделение ответственности

```
Трекер      — ЧТО делать
Роль        — КАК действовать
Оркестратор — КОГО и КОГДА запускать
CI          — прошёл ли результат объективные проверки
Человек     — приоритеты, компромиссы, security, prod
```

## Минимальная команда

| Роль | Делает | Ключевое ограничение |
|------|--------|---------------------|
| **Planning Agent** | цель, scope, acceptance criteria, декомпозиция | не принимает продуктовых решений вне постановки |
| **Developer Agent** | код, тесты, PR | запрещено ослаблять тесты, менять AC, несвязанный рефакторинг |
| **Correctness Reviewer** | требования, границы, достаточность тестов | не изменяет код, новый контекст |
| **Architecture Reviewer** | границы модулей, зависимости, контракты | не изменяет код, новый контекст |
| **Test & Integration** | E2E, staging, логи, отчёт о дефектах | возвращает задачу или эскалирует |
| **Incident Investigator** | локализация по логам/телеметрии, issue | не чинит сам, передаёт разработчику |

Роль — постоянная должностная инструкция: что получает, источники истины,
инструменты, что может/запрещено, обязательные проверки, формат результата,
критерии **PASS / FAIL / BLOCKED**. Конкретная задача передаётся отдельно
(спецификация: цель, scope, AC, ограничения, ожидаемые тесты).

Reviewer запускается в **новом контексте** и не имеет права изменять
проверяемый код. Ревьюеры работают параллельно, замечания объединяются,
цикл `review → fix → review` — до прохождения или лимита.

## Workflow (12 шагов)

Трекер — основная панель управления и state machine:

```
Backlog → Specification → Ready → Development → Agent Review
→ Integration → Human Review → Deploy → Done
```

1. Задача из трекера → 2. Planning создаёт спецификацию → 3. другой агент
проверяет её на полноту → 4. worktree/контейнер на задачу → 5. Developer
реализует → 6. параллельные ревьюеры → 7. исправления → 8. цикл до
прохождения или лимита → 9. CI (детерминированные проверки) → 10. staging →
11. интеграционные/E2E → 12. human approval → deploy → мониторинг → новые задачи.

**Definition of Done:** AC выполнены; тесты добавлены; unit/контрактные/
интеграционные/архитектурные проверки проходят; в staging нет новых ошибок;
ревьюеры не нашли блокирующих; PR содержит проверяемые результаты.
ИИ не решает сам, что «достаточно хорошо» — это определяют тесты, контракты
и политики workflow.

## Harness

```
репозиторий + инструкции + инструменты + sandbox + worktree
+ workflow + тесты + CI + логи + права + retry + human approval
```

Хороший harness: воспроизводимая среда; ограниченные полномочия; короткий
цикл обратной связи; машинно проверяемый результат; сохранение состояния;
наблюдаемость; безопасное восстановление после ошибки.

Критические архитектурные правила закрепляются **кодом в CI** (запрещённые
зависимости, циклические импорты, схемы API, границы модулей). Architecture
Reviewer интерпретирует сложные нарушения — базовые инварианты проверяет CI.

## Harness-фреймворки

Факты сверены с первоисточниками 02.09.2026.

| Фреймворк | Что | Когда брать |
|-----------|-----|-------------|
| **[OpenAI Symphony](https://openai.com/index/open-source-codex-orchestration-symphony)** (апрель 2026, Apache 2.0) | issue tracker как control plane; WORKFLOW.md (YAML + промпт-шаблон); per-issue workspace; retry/reconcile; agent-agnostic | основа минимального harness; +500% landed PR у команд OpenAI. Предупреждения: vague tickets → low-quality PRs; «low-key engineering preview», не production; не-Linear/не-Codex требует адаптеров |
| **[OpenHands Software Agent SDK](https://docs.openhands.dev/sdk)** | Python/TS/REST; 4 пакета; LocalWorkspace → DockerWorkspace → RemoteAPIWorkspace без смены кода агента | самая готовая открытая база исполнителей и reviewer-агентов; sandbox; production |
| **[CrewAI](https://docs.crewai.com/)** | Crews (автономное взаимодействие) + Flows (детерминированный порядок) | быстрое прототипирование ролевых команд; worktree/CI/PR интегрировать самому |
| **[LangGraph](https://langchain-ai.github.io/langgraph/)** | граф состояний: typed state, conditional edges, durable checkpoints, interrupt/resume, HITL | свой строгий оркестратор; нужны пауза/резюме/audit trail |
| **[AutoGen AgentChat](https://microsoft.github.io/autogen/)** | готовые паттерны: RoundRobin, Selector, Swarm (handoff), граф | эксперименты с ролями; для dev-loop нужен свой harness; в maintenance mode → agent-framework |
| **[mini-swe-agent](https://github.com/SWE-bench/mini-swe-agent)** | ~100 строк Python, только bash, >74% SWE-bench verified | компактный исполнитель внутри workflow; НЕ оркестратор |

## Практическая лестница

**1. Самый простой путь** (без оркестратора):
GitHub Issues + один coding agent + worktree на задачу + два reviewer-агента
+ GitHub Actions + ручные статусы.

**2. Следующий уровень:** Linear/Issues + Symphony-подобный оркестратор +
Codex/OpenHands исполнители + Actions + staging + observability.

**3. Своя платформа:** LangGraph/CrewAI Flows + OpenHands SDK + tracker API +
worktree manager + CI/CD + своя permission policy.

Рациональная архитектура: Symphony-модель управления задачами + coding
agent-исполнитель + обычный GitHub CI как источник истины. **Не начинать**
с AutoGen/CrewAI-команды, свободно обсуждающей задачу — надёжнее
фиксированный конвейер с независимым ревью и детерминированными тестами.

## Связь с экосистемой Vector

| Навык/проект | Как соотносится |
|---|---|
| [delegation-gate-checklist](https://github.com/Osmosy/vector-pilot) | 4 шлюза перед spawn = ограниченные полномочия |
| kanban-orchestrator / kanban-worker | Symphony-модель на Hermes (task board как control plane) |
| doubt-driven-review | независимый ревью в свежем контексте |
| [Vector Agent-Ready](https://github.com/Osmosy/vector-agent-ready) | смежное: подготовка контента для агентов |

## Состав репозитория

| Файл | Содержание |
|------|------------|
| `README.md` | Методология (этот документ) |
| `organization-ai-team-source.md` | Исходный конспект (сентябрь 2026) |
| `skills/` | Навык Hermes (операционное ядро) |

## Статус

Методология, не продукт. Факты о фреймворках сверены 02.09.2026.

## Автор

Osmosy. Факты о фреймворках сверены с официальными источниками
(OpenAI Blog, OpenHands Docs, CrewAI, LangGraph, AutoGen, mini-swe-agent).

## Лицензия

MIT — см. [LICENSE](LICENSE).