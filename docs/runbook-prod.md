---
type: architecture
status: draft
project: landing
created: 2026-10-09
---

# Production Runbook — Iberdrola Happy Time

Лендинг разворачивается отдельным Compose-проектом на сервере центрального
Traefik. Репозиторий: https://github.com/luckyfoxdesign/iberdrola-happy-time.
Edge-прокси: `dev/traefik-apps`, серверный каталог `~/traefik-apps`.
Лендинг: `~/iberdrola-happy-time`. Воркспейс на сервер не переносится.

Выбран домен **`iberdrola.luckyfox.design`**. Он задан по умолчанию в
production Compose и как `site` в Astro.

Первое развёртывание выполнено 2026-10-09 из коммита `2b320da`.
Проверки: [production deployment](product/architecture/production-deployment.md).

## Предварительные условия

- Выбран реальный домен; A/AAAA указывают на сервер и соответствуют его сети.
- Центральный Traefik работает; external-сеть `traefik-public` существует.
- Edge использует `websecure`, resolver `le`, TLS options `default` и
  middleware `default-chain@file`; HTTP перенаправляется на HTTPS.
- Порты 80/443 уже обслуживает Traefik. Лендинг не публикует host ports.
- Проверенный код отправлен обычным push в GitHub. Не редактировать tracked
  файлы непосредственно на production.

Сервер из edge runbook: `all-apps-ubuntu-4gb-nbg1-2-sudouser`, пользователь
`sudouser`. Его доступность и DNS нужно проверить перед live-деплоем.
Не менять apex `luckyfox.design`, firewall, SSH или конфигурацию edge TLS.
Секреты лендингу не нужны; `LANDING_DOMAIN` — необязательный override Host.
В примерах он передаётся через shell для HTTP-проверок.

## Pre-deploy — локально

```bash
bash scripts/verify.sh
cd landing
bash scripts/verify.sh
docker compose --env-file /dev/null -f compose.prod.yml config --quiet
docker compose --env-file /dev/null -f compose.prod.yml build
```

Проверить конкретный исходящий diff на секреты и debug-остатки. В новом
репозитории просмотреть только публичные файлы; не делать массовый add
непроверенных путей. После первого коммита: `git diff --check` и
`git diff origin/main -- src docker-files compose.yml compose.prod.yml package.json`.

Production smoke-check без подключения к edge:

```bash
docker run -d --name iberdrola-happy-time-smoke --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m,uid=101,gid=101 \
  --cap-drop ALL --security-opt no-new-privileges:true \
  -p 127.0.0.1:18080:8080 iberdrola-happy-time:local
curl -fsS http://127.0.0.1:18080/healthz
curl -fsS http://127.0.0.1:18080/ > /tmp/iberdrola-happy-time-smoke.html
docker stop iberdrola-happy-time-smoke
docker rm iberdrola-happy-time-smoke
```

Ожидается `ok`, HTML с заголовком, 7 строками и 168 часовыми ячейками.
CSS должен загружаться из `/_astro/`. Неизвестный URL должен отдавать 404.

## Deploy — сервер

Начать с read-only проверки: история приложения, список контейнеров и наличие
сети. Не читать `.env`, сертификаты или credentials.

```bash
ssh all-apps-ubuntu-4gb-nbg1-2-sudouser
```

Первое развёртывание после обычного push:

```bash
git clone https://github.com/luckyfoxdesign/iberdrola-happy-time.git ~/iberdrola-happy-time
cd ~/iberdrola-happy-time
export LANDING_DOMAIN=iberdrola.luckyfox.design
docker network inspect traefik-public --format '{{.Name}}'
docker compose --env-file /dev/null -f compose.prod.yml config --quiet
docker compose --env-file /dev/null -f compose.prod.yml up -d --build --wait
```

Повторное развёртывание:

```bash
cd ~/iberdrola-happy-time
export LANDING_DOMAIN=iberdrola.luckyfox.design
git status --short
git log -1 --oneline
git pull --ff-only
docker compose --env-file /dev/null -f compose.prod.yml config --quiet
docker compose --env-file /dev/null -f compose.prod.yml up -d --build --wait
```

`--env-file /dev/null` предотвращает загрузку серверных `.env`. Без override
Compose использует `iberdrola.luckyfox.design`; переменная в командах выше
также задаёт адрес для `curl`.
В `traefik-apps/compose.yml` добавлять сервис не нужно: Docker provider
обнаруживает labels контейнера лендинга в общей сети.

## После публикации

```bash
docker compose --env-file /dev/null -f compose.prod.yml ps
curl -fsS "https://${LANDING_DOMAIN}/healthz"
curl -fsS "https://${LANDING_DOMAIN}/" > /tmp/iberdrola-happy-time-published.html
curl -sSI "http://${LANDING_DOMAIN}/"
curl -sSI "https://${LANDING_DOMAIN}/"
```

Проверить health status `healthy`, страницу и CSS, валидный HTTPS-сертификат,
HTTP → HTTPS, отсутствие опубликованных host ports, заголовки
`Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`,
`Referrer-Policy`, `Permissions-Policy`. Smoke-check сам по себе не доказывает
DNS, сертификаты или работу центрального edge.

## Rollback

```bash
cd ~/iberdrola-happy-time
export LANDING_DOMAIN=iberdrola.luckyfox.design
git log -5 --oneline
git switch --detach <previous-good-sha>
docker compose --env-file /dev/null -f compose.prod.yml config --quiet
docker compose --env-file /dev/null -f compose.prod.yml up -d --build --wait
```

Повторить HTTPS/health проверки. Вернуться на рабочую ветку перед следующим
обновлением: `git switch main`, затем `git pull --ff-only` и rebuild.
Данных и именованных volumes у лендинга нет; восстановление БД не требуется.
