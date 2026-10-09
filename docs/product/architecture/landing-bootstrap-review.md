---
type: research
status: done
project: landing
created: 2026-10-09
---

# Проверка начальной конфигурации лендинга

Проверен локальный bootstrap для разработки и последующей публикации.
Это запись самопроверки конкретного snapshot; live-публикация не выполнялась.
Стек Astro + TypeScript + CSS остаётся рабочей рекомендацией до ответа
пользователя. Пользователь выбрал домен `iberdrola.luckyfox.design`.

## Snapshot

- 25 публичных файлов репозитория `landing/`; список и SHA256 ниже.
- Aggregate SHA256 (sorted path + space + file SHA256 + newline): `60946bdfe599a8119871659f07d0a59963e740792625877b9da85069349c52b3`.
- 20 production/config/tooling файлов, 4 документа контекста и README,
  1 файл тестов. Начальный budget в `landing/AGENTS.md` соблюдён.
- Три direct npm dependency, exact versions и lockfile; registry compatibility
  проверена. Production-образ содержит только nginx и статические файлы.
- Исходный `docs/backlog/todo/start.html` сохранён;
  SHA256 `fbe2ce4bc44c21d41198539953216476cb00a5bb3ac2ae142e58d18999823e6a`.

## Evidence

| Проверка | Результат |
|---|---|
| `bash scripts/verify.sh` в workspace | PASS: context, workflow ownership, frontmatter, task contract, 15 regression tests |
| `bash scripts/verify.sh` в landing, после финального изменения config | PASS: Compose, 3 тарифных теста, Astro check: 0 ошибок/предупреждений, static build |
| `docker compose --env-file .env.example -f compose.prod.yml build` | PASS: typecheck и production-сборка |
| HTTP smoke на `127.0.0.1:18080` | PASS: `/healthz`, HTML, 7 строк, 168 ячеек, все тарифы и aria-label, CSS/immutable cache, unknown URL 404 |
| Runtime container | PASS: user `101:101`, read-only filesystem, cap drop `ALL`, wget healthcheck |
| Playwright Chromium в Docker, 375 и 1440 px | PASS: bounding boxes, цвета и 168 tooltip совпадают с исходным HTML; горизонтального overflow и page errors нет |
| Dev HTTP | PASS: страница на `http://localhost:3000` |
| CSS source | PASS: CSS побайтово совпадает с извлечённым style исходного HTML |
| Git whitespace review | PASS: `git diff --check`; отдельный просмотр новых публичных файлов |

Browser-проверка выполнена transient Playwright `1.63.0`, совпадающим с
существующим Docker browser image; зависимость в проект не добавлялась.
Скриншоты: `/tmp/iberdrola-happy-time-preview/landing-375.png` и
`/tmp/iberdrola-happy-time-preview/landing-1440.png`.

## Review

Проверены исходники, package/config/Docker файлы, dev/prod команды и карты.
Подтверждённых blocker/high в локальной конфигурации не найдено.
Production router использует `iberdrola.luckyfox.design` по умолчанию;
`LANDING_DOMAIN` — необязательный override. Placeholder используется
только для локальной валидации override. Runtime не публикует host ports,
использует `traefik-public`, HTTPS `websecure`, resolver `le`, TLS options
`default` и `default-chain@file`. Edge TLS/security config не изменялась.
Секреты в build context не включаются; локальные Compose-команды не читают `.env`.
Существующие сторонние изменения в `traefik-apps` сохранены.

Live DNS/HTTPS/Traefik smoke не проверялся: нужны проверка DNS выбранного
домена и публикация проверенного кода в GitHub. Коммитов, push и изменений сервера не выполнялось.
Локальный dev-сервис оставлен запущенным; временный nginx smoke-контейнер удалён.

## Follow-up: выбранный поддомен

Пользователь выбрал `iberdrola.luckyfox.design`. Изменены 4 production/config
файла: Astro `site`, default Host в Compose, дополнительная проверка default
Compose в gate и комментарии `.env.example`. Новых зависимостей нет.
Обновлены инструкции публикации и запись подключения в `traefik-apps`.

Проверки после изменений: PASS — `bash landing/scripts/verify.sh` (default и
placeholder Compose, 3 теста, typecheck без ошибок/предупреждений, build),
production build с `--env-file /dev/null`, `git diff --check` во всех трёх repos.
Production nginx image config SHA256 остался
`686def5dc0ac6ea35fd39dbea111f17078e643d00689df836fa139b8644fe775`:
конечный HTML/CSS/runtime не изменились, поэтому исходные HTTP и visual
проверки переиспользованы. Snapshot и хэши ниже обновлены на текущие файлы.

Фокусированный review: default domain и Astro site совпадают, `.env` не нужен,
существующие TLS/security labels сохранены. Подтверждённых blocker/high нет.
Live-деплой, DNS, git push и серверные изменения в этом follow-up не выполнялись.

## File hashes

```text
.dockerignore 01a4064d903cd2eabc51597c25ea82d93944c0a41aff77eaeb1b9e9129dfa8b2
.env.example 4cae0ac7711cd10864f2320bbb9ee33ac03083c666f84213a8caf0c2e8d2828e
.gitignore 2481653bae3c45e5123b80740a20e00f2e5ef799097954b8f2c9077d989ca89b
AGENTS.md aeaafebef95386ff391e77ba75fea27a296ce4a8d3b54d04a084545f9c1d5c97
CLAUDE.md 05b3c72f1ecbbbe42879c37c8c1730a250fbe9ce2827eaa15b10c8013860a65e
PROJECT_MAP.md e882f8f5931a338ff79ebe5474512f756e7ded4dc446595b1c9e1f0e5c263c13
README.md d1607831bd738cc3a1281abd1ef0b7ce214e6b265fbb4707c2047302dcadc75a
astro.config.mjs ec7e154ba923ff9d4bb9a788ad3e62ce28a863fa2e5cfd2cb7d935a149b75330
compose.prod.yml 90a5cb0ca56db27814e706fad6d2190de7da6d66d2debab4e11fd6749632b7ec
compose.yml 2481120edc199a0cf32e434e1a09ca6b4f6975a2cfee3bf8a8f5de0e1bf39e68
docker-files/Dockerfile.dev 0ddb2a802205acf4116563d2fdd00d4deb35f9c516d716e39670f034377effc0
docker-files/Dockerfile.prod 105943411474d203a0246036bdb465ad1c436f30a4251892e866b1663da2f812
docker-files/nginx.conf 59828ae747d8e8b0d4d95b28b794a9c3f119b47338b2f92672ca1ff991004ccc
package-lock.json c5a46db15812ad206d64e5bb1f02192e066154cab2162e214217fc322ff705e4
package.json ba2dae346151d24eddaef6c6aead7b0791bbc4eef0cba548094660eca11f5592
scripts/update-project-map.sh 8d9fb8e99cb52ea7f1171466c28b6b69ce6e844369b4ea17c40de3775d7388cc
scripts/verify.sh 0e6acfa690433d26c15873421d465958cfccdeb392152675d84b1a2124ddef7a
src/components/PriceCards.astro df832978c3676204678adadb07c71c7a04a998fab60e70d3f993b24b13955b2e
src/components/TariffSchedule.astro 88910f6da2d2dd3c781c81b4ed8ba6c7781222c03c9b6ca8aa134211825c714c
src/data/tariffs.ts 10698abb3260b857ad6cc8156cbe0210ff6a890ea25fd346bd293dd4b3dc9377
src/layouts/BaseLayout.astro c2a7ed02380e539e44286f4d0c4f09da9012a78f377774ab57c7a2c1ff42b99d
src/pages/index.astro 8c25b3219d0d116f4b28dd3478997f915bf7a2b1ad0e3f68a46013673c991276
src/styles/global.css ebf004ef85eef73211a67618f864990dd7f247dc3e2d3af33687a4df17adfaaf
tests/tariffs.test.ts 613fb87b3c2b28fb824acaf82b598bf857a7e64a801e613d9d7dd0889288e63f
tsconfig.json 31b1168d539a42e5dcb309375190602f8724f455374dee28bc9f22de57c909e4
```
