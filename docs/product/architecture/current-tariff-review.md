---
type: research
status: done
project: landing
created: 2026-10-09
related_code:
  - landing/src/components/TariffSchedule.astro
  - landing/src/data/current-time.ts
  - landing/src/styles/global.css
  - landing/tests/tariffs.test.ts
---

# Текущий тариф и маркер времени Валенсии — review

Запрос пользователя: понимать текущий час и тариф сразу при взгляде на heatmap,
с учётом проживания в Валенсии. Реализовано на существующем недельном расписании.

## Изменения и бюджет

Изменены 3 production файла: компонент сетки, CSS и новый чистый helper времени.
Обновлены тесты и контекстные документы; npm-зависимости не менялись.
Default budget в 5 production файлов соблюдён.
Сетка остаётся статическим HTML; небольшой browser script добавляет статус,
уникальную метку текущего часа, подпись «Сегодня» и актуальную шкалу.

Время: `Intl.DateTimeFormat` с `Europe/Madrid`, без геолокации и сетевых API.
Следующие переходы вычисляются по реальным UTC-инстантам, поэтому повторный
и пропущенный час DST учитываются в оставшемся времени.
Clock API и возврат во вкладку:
[DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat),
[visibilitychange](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event).

## Проверки результата

- [x] Текущий тариф, цена, диапазон, следующее переключение и ближайшее дешёвое время видны над сеткой.
  Evidence: production browser smoke и просмотр screenshot 375 px.
- [x] Время Валенсии независимо от часового пояса устройства.
  Evidence: 6 фиксированных UTC-инстантов в Chromium с `America/Los_Angeles`.
- [x] Ровно одна текущая ячейка, `aria-current`, правильный день и подпись «Сегодня».
  Evidence: browser assertions для зимы, лета, полуночи и двух повторных 02:30.
- [x] Автоматическое обновление без reload.
  Evidence: fake-clock advance через 10:00 и полночь; visibilitychange после смены system time.
- [x] DST не искажает countdown.
  Evidence: тесты выходных с пропущенным/повторным часом; повторные осенние 02:30 в browser.
- [x] Мобильная вёрстка без горизонтального overflow; подписи часов не сливаются.
  Evidence: Chromium 320/375/1440 px и assertions скрытия соседнего tick.
- [x] Без JavaScript сохраняется сетка и пояснение, текущий статус не выдаётся за статически известный.
  Evidence: browser context `javaScriptEnabled: false`, 168 ячеек, visible noscript, hidden current card.

`bash landing/scripts/verify.sh`: PASS — Compose, 8 тестов, typecheck
(7 файлов, 0 ошибок/предупреждений/hints), static build.
Production Docker build: PASS. `/healthz`: `ok`; localhost dev: HTTP 200.
`git diff --check` в landing: PASS.
Transient browser script: `/tmp/iberdrola-happy-time-preview/current-tariff-check.mjs`;
Playwright `1.63.0` установлен только во временном Docker-контейнере.
Скриншоты: `current-tariff-320.png`, `current-tariff-375.png`, `current-tariff-1440.png`
в `/tmp/iberdrola-happy-time-preview/`.

## Review итогового snapshot

Самопроверка исходников, DOM-обновлений, таймера и соответствия расписанию выполнена.
Подтверждённых blocker/high не найдено. Текст вставляется через `textContent`;
нет внешних запросов, доступа к приватным данным или новых зависимостей.
Live region объявляет смену тарифного заголовка, часы не озвучиваются каждую минуту.
Таймер выровнен по началу минуты и пересоздаётся при возвращении во вкладку.

Тарифы и праздничные исключения остаются в рамках исходного недельного макета:
календарь национальных праздников не добавлялся, исходная заметка сохранена.
Production TLS/security config не изменялась. Deploy и Git push не выполнялись.
Dev-сервис оставлен запущенным на localhost:3000; временный smoke-контейнер удалён.

Aggregate SHA256 для 26 публичных файлов repo (sorted path + space + SHA256 + newline):
`992a818472a9f8a5dc3b7d057857765cf9114c032a92cbb6c09801ed888821d7`.

## File hashes

```text
.dockerignore 01a4064d903cd2eabc51597c25ea82d93944c0a41aff77eaeb1b9e9129dfa8b2
.env.example 4cae0ac7711cd10864f2320bbb9ee33ac03083c666f84213a8caf0c2e8d2828e
.gitignore 2481653bae3c45e5123b80740a20e00f2e5ef799097954b8f2c9077d989ca89b
AGENTS.md aeaafebef95386ff391e77ba75fea27a296ce4a8d3b54d04a084545f9c1d5c97
CLAUDE.md 83e1b87d57591b011844592715b2af08407ca687ef47e175ab7806ea534c0014
PROJECT_MAP.md d4cbc2afa925fc0b5c91fb38fab9ecb013a2709c23afcf588a7099983f97822c
README.md 7d1b984a9992a349d2f257ebd01d78ca396c3c21ba896ac017d480b232083018
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
src/components/TariffSchedule.astro 8d7c463882879ac0d849d5a63d9de9beb1174e9ec3f394ae7abd5fd3df302c09
src/data/current-time.ts 60854a97ce6edc4e8e293b9876033a714075197857caa41942694173b2d78271
src/data/tariffs.ts 10698abb3260b857ad6cc8156cbe0210ff6a890ea25fd346bd293dd4b3dc9377
src/layouts/BaseLayout.astro c2a7ed02380e539e44286f4d0c4f09da9012a78f377774ab57c7a2c1ff42b99d
src/pages/index.astro 8c25b3219d0d116f4b28dd3478997f915bf7a2b1ad0e3f68a46013673c991276
src/styles/global.css ae683fe56d51ac40c2f0797e76bc3f6863724f843a34aba772f8cc50e3f29555
tests/tariffs.test.ts 17a8eb8c3c83d7d805865df18207f70313ae4f4d2909f0e2a35b660ee148004a
tsconfig.json 31b1168d539a42e5dcb309375190602f8724f455374dee28bc9f22de57c909e4
```
