---
name: vector-ai-team
description: Use when organizing AI agent teams — roles and workflow
version: 1.0.0
author: Osmosy
license: MIT
metadata:
  hermes:
    tags: [agents, orchestration, harness, workflow, roles, development]
    related_skills: [delegation-gate-checklist, kanban-orchestrator, doubt-driven-review, hermes-agent]
---

# Vector AI Team — организация разработки командой ИИ-агентов

Канонический источник: репо github.com/Osmosy/vector-ai-team
(локально ~/projects/vector-ai-team/README.md — полная методология с
таблицами фреймворков и лестницей внедрения). Этот файл — операционное ядро.

## Главный принцип

Главное — не промпты персонажей, а исполняемые ограничения. Независимость
агентов = отдельные контексты/полномочия/критерии, не разные модели.

Трекер — ЧТО делать. Роль — КАК. Оркестратор — КОГО и КОГДА. CI — прошёл ли.
Человек — приоритеты, компромиссы, security, prod.

## Минимальная команда (4 функции)

1. Planning Agent: цель, scope, AC, декомпозиция. Не принимает продуктовых
   решений вне постановки.
2. Developer Agent: код, тесты, PR. Запрещено ослаблять тесты, менять AC,
   несвязанный рефакторинг.
3. Reviewers (Correctness + Architecture, параллельно, НОВЫЙ контекст,
   без права изменять код): замечания объединяются, цикл до лимита.
4. Test/Integration + Incident Investigator: E2E, staging, логи; дефекты
   возвращаются или эскалируются.

## Workflow (12 шагов)

Трекер = state machine (Backlog → Spec → Ready → Dev → Agent Review →
Integration → Human Review → Deploy → Done). worktree/контейнер на задачу.
CI — детерминированные проверки. Staging + E2E. Human approval перед deploy.

Definition of Done: AC выполнены; тесты добавлены; все уровни проверок
проходят; ревьюеры не нашли блокирующих; PR содержит проверяемые результаты.
ИИ не решает сам, что «достаточно хорошо».

## Harness-фреймворки (проверено 02.09.2026)

- **Symphony** (OpenAI, апрель 2026, Apache 2.0): issue tracker как control
  plane, WORKFLOW.md, per-issue workspace, retry/reconcile. Basis для
  минимального harness; предупреждения: vague tickets, не production,
  не-Linear требует адаптеров.
- **OpenHands SDK**: самая готовая база исполнителей (Local/Docker/Remote
  workspace без смены кода агента).
- **CrewAI / LangGraph / AutoGen**: прототипы, строгие графы, готовые
  паттерны команд (соответственно); для dev-loop нужен свой harness.
- **mini-swe-agent**: компактный исполнитель (~100 строк, bash only), не оркестратор.

## Лестница

1. Без оркестратора: Issues + 1 agent + worktree + 2 reviewers + CI.
2. Symphony-подобный оркестратор + Codex/OpenHands + staging.
3. Своя платформа: LangGraph/CrewAI Flows + OpenHands SDK + permission policy.

НЕ начинать с AutoGen/CrewAI-команды, свободно обсуждающей задачу.

## Связь с Hermes

delegation-gate-checklist (4 шлюза = ограниченные полномочия);
kanban-orchestrator/worker (Symphony-модель на Hermes); doubt-driven-review
(независимый ревью); delegate_task enabled_toolsets (минимальный toolset роли).

## Протокол агент ↔ агент

Как устроена связь между агентами в рабочем продукте (Claude Code 2.1.87), а не в теории —
`references/agent-to-agent-protocol.md`: почтовый ящик на агента, адресация по имени,
`request_id` у каждой просьбы, простой как штатное состояние, разрешения через лида,
задача с общей доски как единица передачи работы, запрет на вложенных тиммейтов.

Читать перед тем, как проектировать собственную схему обмена между агентами: там же —
что из этого мы переносим себе и что осталось непроверенным.

## Verification Checklist

- [ ] У каждой роли запреты и критерии PASS/FAIL/BLOCKED
- [ ] Reviewer в новом контексте, без права изменять код
- [ ] Базовые инварианты в CI, не в агенте
- [ ] AC и ожидаемые тесты ДО разработки
- [ ] Точки human approval определены
- [ ] Лимиты циклов review→fix в workflow
- [ ] Логи/traces после deploy
