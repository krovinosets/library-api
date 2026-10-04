# REVIEW — саморевью Library API

## Основание

##  URL и ресурсная модель

| Проверка | Результат | Где проверять |
| --- | --- | --- |
| Версия в базовом URL | Выполнено | `servers[0].url = http://localhost:4010/v1/`; paths без `/v1` |
| Ресурсы во множественном числе | Выполнено | `/books`, `/readers`, `/loans` |
| Kebab-case URL, snake_case JSON | Выполнено | Фильтры `published-year`, `book-id`, `reader-id`; JSON `published_year`, `book_id` |
| ID в path, фильтры в query | Выполнено | `{id}` + переиспользуемый UUID path-параметр; query на коллекциях |
| Возврат отдельной операцией | Выполнено | `POST /loans/{id}/return`; DELETE Loan отсутствует |
| История после удаления | Выполнено | Логическое удаление Books/Readers; Loan и исторические ID сохраняются |

## Методы, статусы, схемы

| Проверка | Результат | Где проверять |
| --- | --- | --- |
| CRUD Books/Readers | Выполнено | GET списка/id, POST, PUT, PATCH, DELETE: по 6 операций |
| Loans: создание, список, ID, возврат | Выполнено | 4 операции Loans |
| Создание 201 + Location | Выполнено | POST трёх коллекций |
| Удаление 204 без тела | Выполнено | DELETE Books/Readers; отсутствует content у 204 |
| 400/404/409/422 не заменены 200 | Выполнено | Явные responses + единый ErrorResponse; дополнительно 415 |
| Request/response раздельные | Выполнено | Create/Replace/UpdateRequest и Book/Reader/Loan |
| Серверные поля запрещены в запросах | Выполнено | Не входят в request properties; additionalProperties=false |
| PATCH непустой, PUT полный | Выполнено | minProperties=1 и required полного набора соответственно |
| Типы/required/enum/format/null | Выполнено | UUID, email, даты, integer, status, коды ошибок; returned_at = string/null |
| Все бизнес-правила | Выполнено | POST /loans, возврат, DELETE, PUT/PATCH Books |
| available_copies вычисляется | Выполнено | total_copies − число активных Loan; min 0, readOnly |
| Изменение total_copies безопасно | Выполнено | 409, если меньше активных выдач; проверка атомарна |

## Ошибки и коллекции

| Проверка | Результат | Где проверять |
| --- | --- | --- |
| Один ErrorResponse | Выполнено | components.schemas.ErrorResponse, components.responses |
| Все шесть обязательных error.code | Выполнено | enum без переименований |
| Пример 422 с details | Выполнено | InvalidDueDate, UnknownField, ValidationError, InvalidIsbn |
| Фильтры на всех коллекциях | Выполнено | q/author/isbn/published-year/available; q/email/registered-from/to; status/book-id/reader-id/due-from/to |
| Сортировка на всех коллекциях | Выполнено | enum sort, направление, tie-break id ASC |
| Пагинация на всех коллекциях | Выполнено | Limit/Offset, Pagination и ListResponse; default/max заданы |
| Просроченные | Выполнено | `GET /loans?status=overdue`; точная формула по UTC |
| Пустой список | Выполнено | 200 + data=[]; отсутствующий ID-фильтр не даёт 404 |
| Запрет выдачи при нуле экземпляров | Выполнено | 409 BUSINESS_RULE_VIOLATION/no_available_copies |
| Лимит 5 активных | Выполнено | 409 BUSINESS_RULE_VIOLATION/active_loans_limit; overdue входит в active |
| Идемпотентность | Выполнено | required header, scope, fingerprint, replay байтов/статуса/заголовков, mismatch, in-progress, TTL |
| Граница TTL и сбои исполнителя | Выполнено | 24 часа от completion; in-progress не удаляется по этому TTL; восстановление без дублирования |
| Конкурентные разные ключи | Выполнено в контракте | Атомарные проверки по книге и читателю; одной идемпотентности недостаточно |

## Документация и техническая проверка

| Проверка | Результат | Где проверять |
| --- | --- | --- |
| Tags Books/Readers/Loans | Выполнено | Все 16 операций |
| summary/description | Выполнено | Все операции |
| Уникальные operationId | Выполнено | Все 16 операций |
| Минимум 3 examples, включая ошибку | Выполнено | 18 именованных examples плюс примеры схем и requestBody |
| `$ref` разрешаются | Проверено локально | Проверка всех внутренних ссылок в YAML |
| YAML разбирается | Проверено локально | safe_load и равенство исходной структуре |
| Схемы JSON Schema 2020-12 | Проверено локально | check_schema для каждого компонента |
| Примеры проходят схемы | Проверено локально | JSON Schema с проверкой format; request/response/component examples |
| Негативные schema-тесты | Проверено локально | unknown/server fields, пустой PATCH, null в update, enum, limit |
| Внешний OpenAPI lint | Не запускался в этой среде | Команда Redocly в README; это не выдано за выполненную проверку |
| Mock/реальный backend | Не реализован и не запущен | Задание про контракт; серверный код в архив не входит |

## Рубрикатор задания

| Критерий | Самооценка | Доказательство |
| --- | --- | --- |
| 1. REST | Полностью в контракте | CRUD + Loans, 201/204/404/409/422, /v1/ |
| 2. Схемы | Полностью в контракте | Раздельные request/response, closed schemas, computed field |
| 3. Ошибки/списки | Полностью в контракте | Единый формат, фильтры, sort, limit/offset, overdue |
| 4. Идемпотентность/правила | Полностью в контракте | Политика POST /loans и атомарность бизнес-операций |
| 5. Документация | Полностью | Tags, описания, именованные examples |
| 6. Сопроводительные файлы | Полностью | STYLE.md (10 правил), REVIEW.md, README.md |
| 7. Бонус | Полностью | Таблица совместимых/несовместимых изменений для v2 в README.md |

