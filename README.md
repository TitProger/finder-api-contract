OpenAPI-контракт API Finder (**Contract First**).

Единственный источник правды по HTTP API для бэкенда и фронтенда.
Реализации пинятся на **git tag**, не на «последний main».

## Связанные каталоги

| Каталог | Роль |
| --- | --- |
| **Finder-api-contract** (этот) | OpenAPI, версии (tags) |
| **Finder-backend** | `CONTRACT_VERSION` + `npm run contract:sync` |
| **Finder-frontend** | тот же tag и свой `npm run contract:sync` |

Remote у этого каталога нет: тег локальный.

## Структура

```
.
├── openapi.yaml
├── paths/
│   ├── common/health.yaml
│   ├── category/categories.yaml
│   ├── inquiry/inquiries.yaml
│   ├── market/optimal-cards.yaml
│   └── research/ingest.yaml
├── components/
│   ├── securitySchemes.yaml
│   ├── parameters/common.yaml
│   ├── schemas/
│   │   ├── common.yaml
│   │   ├── category/category.yaml
│   │   ├── inquiry/inquiry.yaml
│   │   ├── market/optimal-card.yaml
│   │   └── research/dossier.yaml
│   └── responses/errors.yaml
├── CHANGELOG.md
└── README.md
```

Новая доменная область = новый файл в `paths/` + схемы в `components/schemas/` + `$ref` в корневом `openapi.yaml`.

## Конвенции

- OpenAPI **3.1**. Корневой `openapi.yaml` только собирает `$ref`; тела операций и схемы лежат в файлах домена.
- `operationId` — camelCase (`getHealth`, `createCategory`, `getInquiryById`).
- Теги текущих ручек: `health`, `categories`, `inquiries`, `optimal-cards`, `research`. Имена тегов не совпадают с папками `common`, `category`, `inquiry`, `market`: так уже сгенерированы типы бэкенда, переименовывать теги нельзя тем же деревом.
- Успешный JSON — `allOf` из `ApiResponse` и `data`. Ошибки — `$ref` на `components/responses/errors.yaml` (тот же конверт).
- Общие query для будущих списков: `page`, `limit`, `sort`. В ответе списка — `page` (`PageInfo`). Текущие ручки эти параметры не подключают.
- `bearerAuth` — статический `ADMIN_API_TOKEN`. `researchIngestToken` — секрет `POST /research/ingest`. Глобального `security` нет: публичность задаёт сама операция (`GET /health` — `security: []`).
- Описания — на русском. Имена схем — PascalCase, поля — camelCase.

## Версионирование

1. Изменение контракта попадает в `main` этого каталога.
2. Tag локально:

```bash
git tag v0.8.0
```

Remote нет, tag не пушится.

3. Кратко описать изменения в `CHANGELOG.md` (Breaking — явно).

Семантика:

- **patch** — правки описаний, примеры
- **minor** — новые эндпоинты / опциональные поля
- **major** — ломающие изменения (удаление или переименование полей, смена enum)

## Как подключить на бэке

В `.env` backend — локальный путь к этому каталогу:

```env
CONTRACT_REPO=../../Finder-api-contract
```

В корне `backend` файл `CONTRACT_VERSION`:

```text
v0.8.0
```

Затем из `backend`:

```bash
npm run contract:sync
```

Скрипт делает checkout тега, склеивает бандл в `api/openapi.*` и генерирует типы в `src/interfaces/http/generated`. Эти файлы руками не правят.

Фронт пинится на тот же tag: `CONTRACT_REPO=../Finder-api-contract`.

## Локальная проверка бандла

```bash
npx @redocly/cli bundle openapi.yaml -o /tmp/openapi.bundled.yaml
```

## Текущая версия

См. последний tag и `CHANGELOG.md`. Текущая: **v0.8.0**.
