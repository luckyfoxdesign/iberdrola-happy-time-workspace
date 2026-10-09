---
type: architecture
status: draft
project: landing
created: 2026-10-09
---

# Local Development Runbook

Рабочий код находится в независимом репозитории `landing/`:
https://github.com/luckyfoxdesign/iberdrola-happy-time.
Воркспейс хранит инструкции и product docs; он не разворачивается на сервере.

## Разработка

```bash
cd landing
docker compose --env-file /dev/null up --build astro-dev
```

Открыть http://localhost:3000. Стек: Astro, TypeScript и обычный CSS.
Node/npm на хосте не нужны. Изменения в `src/` подхватываются автоматически;
после изменения lockfile нужно пересобрать образ.

```bash
cd landing
bash scripts/verify.sh
bash scripts/update-project-map.sh
```

Gate проверяет Compose, тарифные границы, TypeScript и статическую сборку.
Локальная разработка не требует Traefik, домена или `.env`.
Карта исходников и Docker-сервисов: `landing/PROJECT_MAP.md`.

## Production-сборка

Из `landing/`:

```bash
docker compose --env-file /dev/null -f compose.prod.yml config --quiet
docker compose --env-file /dev/null -f compose.prod.yml build
```

Production Compose по умолчанию использует `iberdrola.luckyfox.design`,
без `.env`. `.env.example` документирует placeholder для необязательного
override `LANDING_DOMAIN`. Live-публикация требует проверки DNS.
Операционные инструкции: [production runbook](runbook-prod.md).
