# F7_API_Contract.md

# F7 — Validation Rules API Contract

## 1. Назначение

API предназначен для управления пользовательскими правилами проверки данных.

Правила привязываются к конкретному датасету и одному его столбцу.

F7 отвечает за создание, получение, изменение, включение/выключение и удаление правил.

Выполнение правил и поиск нарушений в данные в F7 не входят.

## 2. Структура правила

Каждое правило содержит:

| Поле | Тип | Описание |
|---|---|---|
| `id` | integer | ID правила |
| `dataset_id` | integer | ID датасета |
| `column_name` | string | Название столбца |
| `rule_type` | string | Тип правила |
| `params` | object | Параметры правила |
| `is_enabled` | boolean | Включено ли правило |
| `created_at` | string | Дата создания |

## 3. Поддерживаемые типы правил

### not_null

Значение не должно быть пустым.

```json
{
  "rule_type": "not_null",
  "params": {}
}
```

### unique

Значения не должны повторяться.

```json
{
  "rule_type": "unique",
  "params": {}
}
```

### range

Значение должно находиться в заданном диапазоне.

```json
{
  "rule_type": "range",
  "params": {
    "min": 0,
    "max": 120
  }
}
```

`min` и `max` — числовые границы. Обе границы обязательны.

### regex

Значение должно соответствовать регулярному выражению.

```json
{
  "rule_type": "regex",
  "params": {
    "pattern": "^\\S+@\\S+$"
  }
}
```

`pattern` — обязательная строка с шаблоном регулярного выражения.

### allowed_values

Значение должно входить в список разрешённых значений.

```json
{
  "rule_type": "allowed_values",
  "params": {
    "values": ["M", "F"]
  }
}
```

`values` — непустой массив значений.

### data_type

Значение должно соответствовать указанному типу данных.

```json
{
  "rule_type": "data_type",
  "params": {
    "type": "int"
  }
}
```

Допустимые значения `type`:

- `int`
- `float`
- `string`
- `date`

## 4. Создание правила

`POST /api/datasets/{id}/rules`

`{id}` — ID датасета.

Тело запроса:

```json
{
  "column_name": "age",
  "rule_type": "range",
  "params": {
    "min": 0,
    "max": 120
  }
}
```

Успешный ответ: HTTP `201 Created`

```json
{
  "id": 7,
  "dataset_id": 3,
  "column_name": "age",
  "rule_type": "range",
  "params": {
    "min": 0,
    "max": 120
  },
  "is_enabled": true,
  "created_at": "2026-11-27T15:00:00Z"
}
```

`is_enabled` при создании автоматически устанавливается в `true`.

## 5. Получение правил датасета

`GET /api/datasets/{id}/rules`

Успешный ответ: HTTP `200 OK`

```json
{
  "items": [
    {
      "id": 7,
      "dataset_id": 3,
      "column_name": "age",
      "rule_type": "range",
      "params": {
        "min": 0,
        "max": 120
      },
      "is_enabled": true,
      "created_at": "2026-11-27T15:00:00Z"
    }
  ],
  "total": 1
}
```

Если правил нет:

```json
{
  "items": [],
  "total": 0
}
```

## 6. Редактирование правила

`PUT /api/validation-rules/{rule_id}`

Тело запроса содержит обновляемые значения правила:

```json
{
  "column_name": "age",
  "rule_type": "range",
  "params": {
    "min": 1,
    "max": 100
  },
  "is_enabled": true
}
```

Успешный ответ: HTTP `200 OK`.

Возвращается обновлённый объект правила в формате, описанном в разделе 4.

Редактирование позволяет также включать и выключать правило через `is_enabled`.

## 7. Удаление правила

`DELETE /api/validation-rules/{rule_id}`

Успешный ответ: HTTP `200 OK`

```json
{
  "ok": true
}
```

Если правило не найдено, возвращается HTTP `404 Not Found`.

## 8. Ошибки

Все ошибки возвращаются в общем формате проекта.

### Некорректные данные

HTTP `400 Bad Request`

```json
{
  "error": {
    "code": "validation_error",
    "message": "Invalid validation rule data"
  }
}
```

### Датасет, столбец или правило не найдены

HTTP `404 Not Found`

```json
{
  "error": {
    "code": "not_found",
    "message": "Resource not found"
  }
}
```

## 9. Хранение данных

Используется таблица `validation_rules`.

| Поле | Тип | Описание |
|---|---|---|
| `id` | INTEGER | Первичный ключ |
| `dataset_id` | INTEGER | ID датасета |
| `column_name` | TEXT | Название столбца |
| `rule_type` | TEXT | Тип правила |
| `params_json` | TEXT | Параметры в JSON |
| `is_enabled` | INTEGER | 1 — включено, 0 — выключено |
| `created_at` | TEXT | Дата создания |

Параметры правила хранятся в `params_json`, а в API передаются как объект `params`.

## 10. Общие договорённости

- Все запросы и ответы используют JSON.
- Все правила обязательно привязаны к существующему датасету.
- Столбец должен существовать в выбранном датасете.
- Правила разных датасетов не смешиваются.
- Отключённые правила сохраняются, но имеют `is_enabled: false`.
- В F7 не выполняется проверка строк датасета на соответствие правилам.

