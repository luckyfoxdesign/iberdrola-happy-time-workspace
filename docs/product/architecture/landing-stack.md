---
type: architecture
status: draft
project: landing
created: 2026-10-09
related_code:
  - landing/src/
  - landing/compose.prod.yml
---

# Стек лендинга Iberdrola Happy Time

Рабочая рекомендация: **Astro + TypeScript + обычный CSS**. Подготовлен базовый
проект; пользователь может изменить выбор. Макет `docs/backlog/todo/start.html`
содержит одну страницу, карточки тарифов и недельную сетку.

Astro формирует компоненты в статический HTML и позволяет добавлять клиентский
JavaScript по необходимости. Это подходит текущему объёму и будущей вёрстке;
см. [официальное описание Astro](https://docs.astro.build/en/concepts/why-astro/).
TypeScript задаёт типы тарифов и будущих взаимодействий, CSS variables
сохраняют визуальные параметры исходника. Сетка собирается заранее. Небольшой браузерный скрипт отмечает текущий
тариф и час Валенсии (`Europe/Madrid`), показывает следующее переключение
и ближайший дешёвый период; обновляется каждую минуту и при возврате во вкладку.

| Вариант | Когда выбирать |
|---|---|
| Astro + TypeScript + CSS | Статический лендинг с компонентами и отдельными интерактивными блоками |
| Vite + TypeScript + CSS | Максимально прямое развитие исходного HTML и DOM-скрипта |
| React + Vite + TypeScript | Много связанных интерактивных состояний и развитие в полноценное приложение |

## Реализация и совместимость

- Astro `7.3.8`, `@astrojs/check` `0.9.10`, TypeScript `6.0.3`.
- Версии проверены через официальный npm registry в Docker 2026-10-09.
- `@astrojs/check` поддерживает TypeScript `^5.0.0 || ^6.0.0`; поэтому актуальный
  TypeScript 7 не выбран. Совместимость подтверждается `astro check`.
- Node 24 (`24.21.0` в закреплённом образе); Astro требует Node ≥22.12.0:
  [installation docs](https://docs.astro.build/en/install-and-setup/).
- Все npm-версии exact-pinned, зависимости воспроизводятся через lockfile.
- Docker base images закреплены digest; runtime — unprivileged nginx `1.30.5`.

## Публикация

Репозиторий кода: https://github.com/luckyfoxdesign/iberdrola-happy-time,
локальный каталог `landing/`. Статическая сборка отдаётся nginx на внутреннем
порту 8080. Отдельный контейнер подключён к external-сети `traefik-public`.
Router labels включают HTTPS, `le`, TLS options `default`, `default-chain@file`.
Пользователь выбрал `iberdrola.luckyfox.design`: домен задан в Astro `site`
и по умолчанию в production router. `LANDING_DOMAIN` остаётся необязательным
override Host; live-деплой ещё не выполнен.

Конфигурация и модель подключения соответствуют `../traefik-apps/docs/add-new-app.md`.
Product docs остаются в воркспейсе; код публикуется отдельным репозиторием.
Перед публикацией пройти [production runbook](../../runbook-prod.md).
