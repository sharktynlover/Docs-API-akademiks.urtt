#  Документация по API расписания колледжа (akademiks.urtt)

> **Базовый URL:** `https://akademiks.urtt.ru/api/trpc/schedule.get`  

---

## 1. Введение

Данный API позволяет получить расписание занятий для **групп** или **преподавателей** на указанную неделю. Используется в Telegram-боте «Расписание», но может быть полезен и для других студенческих проектов.

API предоставляется в формате **tRPC batch** (один запрос — несколько вызовов, но мы используем только один).

---

## 2. Базовый URL и метод

- **Метод:** `GET`
- **URL:** `https://akademiks.urtt.ru/api/trpc/schedule.get`
- **Параметры (query):**
  - `batch=1` – обязательно (формат tRPC).
  - `input` – закодированный JSON с параметрами запроса.

---

## 3. Параметры запроса (внутри `input`)

Параметр `input` — это URL-закодированный JSON следующей структуры:

```json
{
  "0": {
    "json": {
      "groupId": "bi-132",        // или null
      "teacherId": "alfereva-ov", // или null
      "classroomId": null,        // пока не используется
      "weekStart": "2026-09-06T19:00:00.000Z"
    },
    "meta": {
      "values": {
        "teacherId": ["undefined"],
        "classroomId": ["undefined"],
        "weekStart": ["Date"]
      },
      "v": 1
    }
  }
}
```

### Обязательные поля в `json`:
- **Либо `groupId`, либо `teacherId`** должны быть не `null` (но можно указать оба, тогда вернётся расписание и для группы, и для преподавателя — фактически пересечение).
- `weekStart` – дата начала недели в формате ISO (UTC). Например, `2026-09-06T19:00:00.000Z` соответствует понедельнику 7 сентября 2026 года в Екатеринбурге (UTC+5, поэтому 19:00 UTC = 00:00 +5).

### Необязательные поля:
- `classroomId` – пока не используется, всегда `null`.

**Важно:** Поля `meta` и `v` обязательны, они копируются из примера.

---

## 4. Формирование полного URL

Чтобы получить расписание для группы `bi-132` на неделю, начинающуюся с 2026-09-06, нужно:

1. Создать JSON как в примере выше.
2. Закодировать его в URL-строку (encodeURIComponent).
3. Подставить в `input=`.

**Пример готового URL:**
```
https://akademiks.urtt.ru/api/trpc/schedule.get?batch=1&input=%7B%220%22%3A%7B%22json%22%3A%7B%22groupId%22%3A%22bi-132%22%2C%22teacherId%22%3Anull%2C%22classroomId%22%3Anull%2C%22weekStart%22%3A%222026-09-06T19%3A00%3A00.000Z%22%7D%2C%22meta%22%3A%7B%22values%22%3A%7B%22teacherId%22%3A%5B%22undefined%22%5D%2C%22classroomId%22%3A%5B%22undefined%22%5D%2C%22weekStart%22%3A%5B%22Date%22%5D%7D%2C%22v%22%3A1%7D%7D%7D
```

---

## 5. Примеры запросов

### Для группы `bi-132`
```http
GET /api/trpc/schedule.get?batch=1&input=%7B%220%22%3A%7B%22json%22%3A%7B%22groupId%22%3A%22bi-132%22%2C%22teacherId%22%3Anull%2C%22classroomId%22%3Anull%2C%22weekStart%22%3A%222026-09-06T19%3A00%3A00.000Z%22%7D%2C%22meta%22%3A%7B%22values%22%3A%7B%22teacherId%22%3A%5B%22undefined%22%5D%2C%22classroomId%22%3A%5B%22undefined%22%5D%2C%22weekStart%22%3A%5B%22Date%22%5D%7D%2C%22v%22%3A1%7D%7D%7D
```

### Для преподавателя `alfereva-ov`
```http
GET /api/trpc/schedule.get?batch=1&input=%7B%220%22%3A%7B%22json%22%3A%7B%22groupId%22%3Anull%2C%22teacherId%22%3A%22alfereva-ov%22%2C%22classroomId%22%3Anull%2C%22weekStart%22%3A%222026-09-06T19%3A00%3A00.000Z%22%7D%2C%22meta%22%3A%7B%22values%22%3A%7B%22groupId%22%3A%5B%22undefined%22%5D%2C%22classroomId%22%3A%5B%22undefined%22%5D%2C%22weekStart%22%3A%5B%22Date%22%5D%7D%2C%22v%22%3A1%7D%7D%7D
```

---

## 6. Структура ответа

**Успешный ответ (HTTP 200)** – JSON вида:

```json
{
  "result": {
    "data": {
      "json": {
        "data": [
          // массив занятий
        ],
        "type": "student",   // или "teacher"
        "group": {
          "id": "bi-132",
          "title": "Би-132",
          "additionalId": null
        }
      }
    }
  }
}
```

### Массив `data`
Каждый элемент — объект с полями (примерные):

```json
{
  "dayOfWeek": 1,          // 1=понедельник, 2=вторник, ..., 6=суббота
  "lessonNumber": 1,       // номер пары (1,2,3...)
  "discipline": "Математика",
  "teacherName": "Иванов И.И.",
  "classroom": "365",
  "groupName": "Би-132",
  "subgroup": null,        // если есть подгруппа
  "startTime": "08:30",
  "endTime": "10:05"
}
```

**Примечание:** Точный набор полей может варьироваться, но основные (дисциплина, преподаватель, аудитория, время) всегда присутствуют.

---

## 7. Коды ответов

| Код | Описание |
|-----|----------|
| `200` | Успешно, данные получены |
| `400` | Некорректный запрос (неверный JSON, отсутствие группы/преподавателя) |
| `500` | Внутренняя ошибка сервера |

---

## 8. Идентификаторы групп и преподавателей

### Группы
Используйте **полный идентификатор** (например, `bi-132`). Соответствие между названием группы и ID находится в конфигурации:

```javascript
const GroupConfig = {
  e: { "Э-170":"e-170", "Э-171":"e-171", /* ... */ },
  bi: { "Би-131":"bi-131", "Би-132":"bi-132", /* ... */ },
  // ...
};
```

Полный список можно получить у администратора бота или в исходном коде.

### Преподаватели
Используйте **ID преподавателя** (например, `alfereva-ov`). Маппинг ФИО → ID:

```javascript
const TeacherConfig = {
  "Алексеева О.В.": "alekseeva-ov",
  "Алферьева О.В.": "alfereva-ov",
  // ...
};
```

Полный список также доступен.

---

## 9. Примеры использования

### cURL
```bash
curl "https://akademiks.urtt.ru/api/trpc/schedule.get?batch=1&input=%7B%220%22%3A%7B%22json%22%3A%7B%22groupId%22%3A%22bi-132%22%2C%22teacherId%22%3Anull%2C%22classroomId%22%3Anull%2C%22weekStart%22%3A%222026-09-06T19%3A00%3A00.000Z%22%7D%2C%22meta%22%3A%7B%22values%22%3A%7B%22teacherId%22%3A%5B%22undefined%22%5D%2C%22classroomId%22%3A%5B%22undefined%22%5D%2C%22weekStart%22%3A%5B%22Date%22%5D%7D%2C%22v%22%3A1%7D%7D%7D"
```

### Python (requests)
```python
import requests
import json

url = "https://akademiks.urtt.ru/api/trpc/schedule.get"
params = {
    "batch": 1,
    "input": json.dumps({
        "0": {
            "json": {
                "groupId": "bi-132",
                "teacherId": None,
                "classroomId": None,
                "weekStart": "2026-09-06T19:00:00.000Z"
            },
            "meta": {
                "values": {
                    "teacherId": ["undefined"],
                    "classroomId": ["undefined"],
                    "weekStart": ["Date"]
                },
                "v": 1
            }
        }
    })
}

response = requests.get(url, params=params)
data = response.json()
print(data)
```

### JavaScript (fetch)
```javascript
const params = new URLSearchParams({
  batch: 1,
  input: JSON.stringify({
    "0": {
      "json": {
        "groupId": "bi-132",
        "teacherId": null,
        "classroomId": null,
        "weekStart": "2026-09-06T19:00:00.000Z"
      },
      "meta": {
        "values": {
          "teacherId": ["undefined"],
          "classroomId": ["undefined"],
          "weekStart": ["Date"]
        },
        "v": 1
      }
    }
  })
});

fetch(`https://akademiks.urtt.ru/api/trpc/schedule.get?${params}`)
  .then(res => res.json())
  .then(data => console.log(data));
```

---

## 10. Ограничения и рекомендации

- **Частота запросов:** Не рекомендуется делать более 1 запроса в 10 секунд с одного IP, чтобы не перегружать сервер.
- **Кэширование:** Расписание обновляется нечасто, поэтому сохраняйте ответы локально и обновляйте по расписанию (например, раз в 20 минут).
- **Таймзона:** Все даты в запросе указываются в UTC. Для Екатеринбурга (UTC+5) добавляйте 5 часов к локальному времени начала недели.
- **Обработка ошибок:** Всегда проверяйте статус ответа и наличие поля `result.data.json.data`, чтобы избежать сбоев.

---
