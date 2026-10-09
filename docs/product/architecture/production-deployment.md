---
type: architecture
status: done
project: landing
created: 2026-10-09
related_docs:
  - docs/runbook-prod.md
  - docs/product/architecture/reference-layout-review.md
---

# Первая публикация Iberdrola Happy Time

Опубликован https://iberdrola.luckyfox.design на существующем сервере
`all-apps-ubuntu-4gb-nbg1-2-sudouser`, каталог `~/iberdrola-happy-time`.

**Commits:**
- landing `2b320dade3331207cb2887f47029028a456ff2a8` — "feat: add Valencia tariff landing with live schedule and Traefik deployment".

Код отправлен обычным push в GitHub, ветка `main`; сервер клонировал именно
этот коммит. Исходники соответствуют [проверенному snapshot](reference-layout-review.md).
Production запущен через `docker compose --env-file /dev/null -f compose.prod.yml
up -d --build --wait --wait-timeout 180`. Серверные tracked файлы не редактировались.

## Проверки

- [x] Pre-deploy gates: workspace 15 regression tests; landing 9 unit tests,
  Astro check (0 ошибок/предупреждений/hints), static build и Compose config --quiet.
- [x] Production Docker build локально и на сервере; контейнер
  `iberdrola-happy-time-landing-1` имеет status healthy.
- [x] DNS A — `46.225.23.195`, совпадает с публичным IPv4 сервера; AAAA отсутствует.
- [x] HTTPS главная и `/healthz` — 200; health body — `ok`.
  Проверена цепочка и hostname сертификата Let's Encrypt;
  срок действия до 2027-01-07 13:42 UTC на момент публикации.
- [x] HTTP → HTTPS — 308. Присутствуют HSTS, X-Content-Type-Options,
  X-Frame-Options, Referrer-Policy и Permissions-Policy.
- [x] HTML содержит 7 строк и 168 ячеек; `/_astro/index.5hUgmvyH.css`
  возвращает 200 с immutable cache. Неизвестный URL возвращает 404.
- [x] Browser smoke именно публичного HTTPS сайта: время Валенсии при timezone
  America/Los_Angeles, зима/лето/DST, границы тарифов и полночь, countdown,
  уникальный активный тариф/час, resize, возврат во вкладку и fallback без JS.
  Ширины 320/375/760/1122 px без overflow; ошибок browser нет.
- [x] Runtime: user 101:101, read-only, cap-drop ALL, no-new-privileges,
  пустые host port bindings; подключение только к traefik-public.

Browser evidence: Playwright 1.63.0 во временном Docker-контейнере,
`/tmp/iberdrola-happy-time-preview/published-layout-check.mjs`;
screenshots `published-layout-{320,375,760,1122}.png` в том же каталоге.

## Review и эксплуатация

Подтверждённых blocker/high нет. Deployment повторяет проверенную конфигурацию,
новые production файлы и зависимости не добавлялись. Центральный Traefik,
apex luckyfox.design, firewall, SSH, TLS config и серверные секреты не изменялись.
Секреты и credential-файлы не читались; приложению они не нужны.
Dev localhost:3000 оставлен запущенным.

Обновления и rollback: [runbook](../../runbook-prod.md).
Это первая публикация: предыдущего production коммита для rollback пока нет.
