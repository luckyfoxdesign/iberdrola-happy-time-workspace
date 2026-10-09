---
type: research
status: done
project: landing
created: 2026-10-09
related_code:
  - landing/src/pages/index.astro
  - landing/src/components/PriceCards.astro
  - landing/src/components/TariffSchedule.astro
  - landing/src/styles/global.css
  - landing/src/data/current-time.ts
  - landing/tests/tariffs.test.ts
---

# Вёрстка по визуальному референсу — review

Запрос пользователя: переделать текущий лендинг по приложенному изображению.
Реализованы карточка «Сейчас» с тремя соединёнными тарифными сегментами,
выделение активного сегмента и стрелка, цены под сегментами и два countdown;
отдельная недельная heatmap с легендой, подсветкой сегодняшнего дня и
вертикальным маркером времени; информационная карточка с исходной заметкой.

## Изменения и бюджет

Изменены ровно 5 production файлов: страница, два компонента, CSS и helper времени.
Обновлены тесты и контекстные документы. Default budget соблюдён;
зависимости, тарифные цены и недельное расписание сохранены.
Browser script теперь принадлежит PriceCards; TariffSchedule отдаёт статический HTML.
Время определяется автоматически в Europe/Madrid независимо от timezone устройства.
Countdown показывает реальные ближайшие появления двух остальных тарифов,
а не фиксированные цифры из изображения. Праздничный календарь не добавлялся.

## Проверки результата

- [x] Композиция, цвета, активный сегмент, стрелка, heatmap и заметка соответствуют референсу.
  Evidence: просмотр финальных production screenshots 1122 и 375 px.
- [x] На ширинах 320, 375, 760 и 1122 px нет горизонтального overflow.
  Evidence: Chromium assertions и screenshots всех четырёх размеров.
- [x] Ровно один активный тариф и одна текущая ячейка с aria-current;
  правильный день «Сегодня», два countdown и время Валенсии.
  Evidence: browser assertions для зимы, лета, границы 10:00, субботы
  и двух повторных осенних 02:30; browser timezone America/Los_Angeles.
- [x] Вертикальный маркер учитывает долю часа, корректно перестраивается
  после resize; подпись остаётся внутри графика даже в 23:59.
  Evidence: geometry assertions, resize 375→1122, правый край в 23:59.
- [x] Обновление без reload меняет тариф и день через полночь;
  возвращение во вкладку пересчитывает время.
  Evidence: fake-clock advance 23:59→00:00 и visibilitychange после setSystemTime.
- [x] Без JavaScript видны цены, 168 ячеек и fallback;
  активный тариф и маркер не выдают статические данные за текущие.
  Evidence: context javaScriptEnabled: false; browser errors отсутствуют.
- [x] Helper корректно рассчитывает оба будущих периода, включая выходные и DST.
  Evidence: 9 unit tests, в том числе два точных countdown сценария.

`bash landing/scripts/verify.sh`: PASS — Compose config --quiet,
9 тестов, Astro check (7 файлов, 0 ошибок/предупреждений/hints), static build.
Production Docker build: PASS. Production `/healthz`: `ok`.
Dev localhost:3000: HTTP 200. `git diff --check`: PASS.
Все runtime проверки выполнены в Docker, зависимости browser smoke устанавливались
только во временный контейнер, package.json/lock не изменялись.

Browser script: `/tmp/iberdrola-happy-time-preview/reference-layout-check.mjs`;
Playwright 1.63.0. Финальные screenshots:
`/tmp/iberdrola-happy-time-preview/reference-layout-{320,375,760,1122}.png`.

## Review итогового snapshot

Самопроверка композиции, исходников, DOM-обновлений, геометрии маркера,
таймера и расчёта ближайших периодов выполнена. Подтверждённых blocker/high нет.
Текст обновляется через textContent; сетевых API и новых зависимостей нет.
Live region объявляет тариф, таймер выровнен по началу минуты;
ResizeObserver пересчитывает геометрию и скрывает близкие к линии подписи шкалы.
Исходные цены и оговорка о договоре/праздниках сохранены.
Production TLS/security config не изменялась; deploy и Git push не выполнялись.
Dev остаётся запущенным на localhost:3000.

Aggregate SHA256 для 26 публичных файлов repo
(sorted path + space + SHA256 + newline): `e0a4b0413cc88fcfb36719b4822d003440d57071e5b00b17c60511bc8c5647c2`.

## File hashes

```text
.dockerignore 01a4064d903cd2eabc51597c25ea82d93944c0a41aff77eaeb1b9e9129dfa8b2
.env.example 4cae0ac7711cd10864f2320bbb9ee33ac03083c666f84213a8caf0c2e8d2828e
.gitignore 2481653bae3c45e5123b80740a20e00f2e5ef799097954b8f2c9077d989ca89b
AGENTS.md aeaafebef95386ff391e77ba75fea27a296ce4a8d3b54d04a084545f9c1d5c97
CLAUDE.md 83e1b87d57591b011844592715b2af08407ca687ef47e175ab7806ea534c0014
PROJECT_MAP.md 8c26bd44ab640d776e0a8ff21b1c88038215168ab75972a98d2af562b2419c8b
README.md 78a71faa9acee6e65b37ee255f1377c022e7026fe76c76c2935bd7e2db446040
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
src/components/PriceCards.astro 299243fcd553255a1b2bc0bf98282daec60e00d2bf8ea0bbffe71cef636d9ab0
src/components/TariffSchedule.astro 4f3b1544988e1074c84ed0080be1cd0c3ee5ac1acb55ca69a3097721e8130e11
src/data/current-time.ts b866ddc9c8577d5b60c9cc17cd0842427eed1f40c56c6d08c79cbe6b378bdb0e
src/data/tariffs.ts 10698abb3260b857ad6cc8156cbe0210ff6a890ea25fd346bd293dd4b3dc9377
src/layouts/BaseLayout.astro c2a7ed02380e539e44286f4d0c4f09da9012a78f377774ab57c7a2c1ff42b99d
src/pages/index.astro 6cdd9db6e07b181530d52497d26c0d9928dc359a36c0f489775ac5f171a0e344
src/styles/global.css 38e73d26d99fbc2495ae567ff4d025e3291fa5e44b532d65ca0ee723064867a2
tests/tariffs.test.ts 8eb52fdb6ebfc7d5e56d917e65e96ea1f45fb107cb91d43cfbc9b591f56f9a46
tsconfig.json 31b1168d539a42e5dcb309375190602f8724f455374dee28bc9f22de57c909e4
```
