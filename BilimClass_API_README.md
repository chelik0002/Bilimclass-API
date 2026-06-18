# BilimClass API — Документация

> **Версия документа:** 1.0 · **Источник:** анализ HAR-трафика · **Дата:** июнь 2026  
> Неофициальная документация REST API платформы онлайн-дневника [BilimClass](https://www.bilimclass.kz).

---

## Содержание

- [Общая информация](#общая-информация)
- [Аутентификация](#аутентификация)
  - [Вход в систему](#1-вход-в-систему)
  - [Выход из системы](#2-выход-из-системы)
- [Система](#система)
  - [Учебные годы](#3-получить-список-учебных-годов)
  - [Настройки системы](#4-получить-настройки-системы)
  - [Связанные дети (аккаунт родителя)](#5-получить-список-связанных-детей)
  - [Внешние аккаунты](#6-список-внешних-привязанных-аккаунтов)
  - [Статические модальные окна](#7-получить-список-статических-модальных-окон)
  - [Баннеры](#8-получить-баннеры)
- [Расписание](#расписание)
  - [Расписание уроков на неделю](#9-получить-расписание-уроков-на-неделю)
  - [Список групп/классов](#10-получить-список-группклассов)
- [Электронный дневник и оценки](#электронный-дневник-и-оценки)
  - [Список недель учебного периода](#11-получить-список-недель-учебного-периода)
  - [Дневник за неделю](#12-получить-дневник-за-неделю)
  - [Учебные периоды (четверти)](#13-получить-учебные-периоды-четверти)
  - [Оценки по предметам за период](#14-получить-оценки-по-предметам-за-период)
  - [Итоговые оценки за год](#15-получить-итоговые-оценки-за-год)
- [Домашние задания](#домашние-задания)
  - [Список домашних заданий за месяц](#16-получить-список-домашних-заданий-за-месяц)
- [Чаты](#чаты)
  - [Список доступных собеседников](#17-получить-список-доступных-собеседников)
  - [Активные личные чаты](#18-получить-активные-личные-чаты)
- [Профиль и безопасность](#профиль-и-безопасность)
  - [Проверка смены пароля по умолчанию](#19-проверить-необходимость-смены-пароля)
  - [Список родителей](#20-получить-список-родителей)
  - [Активные сессии](#21-получить-активные-сессии)
  - [Завершённые сессии](#22-получить-завершённые-сессии)
- [Новости (гостевой доступ)](#новости-гостевой-доступ)
  - [Публичные статьи](#23-получить-публичные-статьи-новости)

---

## Общая информация

| Параметр | Значение |
|---|---|
| **Base URL** | `https://api.bilimclass.kz` |
| **Протокол** | HTTPS (HTTP/2) |
| **Формат данных** | JSON (`application/json`) |
| **Версии API** | `v2`, `v4` (одновременно используются) |
| **CORS** | Разрешено только с `https://www.bilimclass.kz` |

### Общие заголовки запроса

Все запросы (кроме публичных) должны содержать следующие заголовки:

| Заголовок | Значение | Описание |
|---|---|---|
| `Accept` | `application/json` | Тип ожидаемого ответа |
| `Content-Type` | `application/json` | Тип тела запроса (для POST) |
| `Authorization` | `Bearer <access_token>` | JWT-токен из ответа `/api/v2/os/login` (для защищённых эндпоинтов) |
| `x-localization` | `ru` / `kk` / `en` | Язык ответа |
| `x-school-id` | `<schoolId>` | ID школы пользователя (для защищённых эндпоинтов) |

> **Примечание:** `schoolId` возвращается в поле `user_info.school_id` при успешной авторизации.

---

## Аутентификация

### 1. Вход в систему

**`POST /api/v2/os/login`**

Аутентификация пользователя по логину и паролю. Возвращает JWT-токен доступа, токен обновления и полную информацию о профиле пользователя.

#### Заголовки

| Заголовок | Значение | Обязателен |
|---|---|---|
| `Content-Type` | `application/json` | ✅ |
| `x-localization` | `ru` | ✅ |
| `x-mathrix` | `<device-uuid>` | ✅ (UUID устройства) |

> **`x-mathrix`** — уникальный идентификатор устройства в формате UUID v4. Генерируется на клиентской стороне.

#### Request Body

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `login` | `string` | ✅ | Логин пользователя |
| `password` | `string` | ✅ | Пароль пользователя |
| `eduYear` | `integer` | ✅ | Учебный год (например, `2025` для 2025–2026 уч. года) |

```json
{
  "login": "ivanov_ivan_1",
  "password": "secret123",
  "eduYear": 2025
}
```

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "success": true,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
  "exp_time": "2027-06-19 01:14:47",
  "refresh_token": "def50200...",
  "user_info": {
    "surname": "Иванов",
    "firstname": "Иван",
    "lastname": "Иванович",
    "bornDate": "01.01.2012",
    "gender": "m",
    "photo": "",
    "position": "Ученик",
    "school": {
      "eduYears": [
        {
          "schoolId": 1000001,
          "eduYear": 2025,
          "isCurrent": true,
          "dateStart": "01.09.2025",
          "dateEnd": "25.05.2026",
          "status": "active",
          "canEditPeriods": false
        },
        {
          "schoolId": 1000001,
          "eduYear": 2024,
          "isCurrent": false,
          "dateStart": "01.09.2024",
          "dateEnd": "25.05.2025",
          "status": "archive",
          "canEditPeriods": false
        }
      ],
      "name": "КГУ «Школа №1»",
      "bin": "000000000000",
      "type": "regular",
      "region": "Северо-Казахстанская область",
      "department": "Петропавловск Г.А.",
      "apiVersion": "v4",
      "flagFiveScoreEnabled": false,
      "scheduleVersion": "v3",
      "journalVersion": "v2",
      "phone": "+77000000000",
      "bilimClassAdminName": "Иванова Анна Ивановна",
      "bilimClassAdminPhone": "+7 (700) 000-00-00"
    },
    "userId": 90000001,
    "defaultPass": true,
    "hasSubscription": true,
    "role": "student",
    "roleVersion": "studentV4",
    "studentInfo": {
      "studentGroupUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
      "studentUuid": "aaaaaaaa-bbbb-cccc-dddd-ffffffffffff",
      "studentSchoolUuid": "aaaaaaaa-bbbb-cccc-dddd-000000000000",
      "studentGroupUuids": [
        "aaaaaaaa-bbbb-cccc-dddd-111111111111"
      ],
      "iin": "000000000000",
      "fullname": "ИВАНОВ ИВАН",
      "hasFreedomCard": false,
      "hasActivatedParent": true
    },
    "school_id": 1000001,
    "group": {
      "id": 900001,
      "name": "7 В",
      "letter": "В",
      "parallel": 7
    },
    "chatToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9..."
  },
  "hash": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `success` | `boolean` | Флаг успешности запроса |
| `access_token` | `string` | JWT-токен (RS256), используется как `Authorization: Bearer <token>` |
| `exp_time` | `string` | Дата истечения токена в формате `YYYY-MM-DD HH:mm:ss` |
| `refresh_token` | `string` | Токен для обновления сессии |
| `user_info` | `object` | Полная информация о пользователе, школе и группе |
| `user_info.chatToken` | `string` | JWT-токен (HS512) для подключения к сервису чатов |

---

### 2. Выход из системы

**`GET /api/logout`**

Завершает текущую сессию пользователя. Токен становится недействительным.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Текущий учебный год |
| `soft` | `integer` | ✅ | Тип выхода: `0` — полный, `1` — мягкий (без инвалидации токена) |

#### Ответ — `200 OK`

```json
{
  "success": true
}
```

---

## Система

### 3. Получить список учебных годов

**`GET /api/v4/os/system/education-years`**

Возвращает все учебные годы, доступные для данной школы, с указанием дат начала и окончания.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Текущий учебный год |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "schoolId": 1000001,
      "eduYear": 2025,
      "isCurrent": true,
      "dateStart": "01.09.2025",
      "dateEnd": "25.05.2026",
      "status": "active",
      "canEditPeriods": false
    },
    {
      "schoolId": 1000001,
      "eduYear": 2024,
      "isCurrent": false,
      "dateStart": "01.09.2024",
      "dateEnd": "25.05.2025",
      "status": "archive",
      "canEditPeriods": false
    }
  ]
}
```

| Поле | Тип | Описание |
|---|---|---|
| `isCurrent` | `boolean` | Является ли год текущим |
| `status` | `string` | Статус года: `active` или `archive` |
| `canEditPeriods` | `boolean` | Разрешено ли редактирование периодов |

---

### 4. Получить настройки системы

**`GET /api/v4/os/system/settings`**

Возвращает набор системных флагов и настроек платформы для данной школы и учебного года. Используется для управления видимостью разделов и функциональностью интерфейса.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": [
    {
      "key": "monitoringTeacherLoadLock",
      "type": "bool",
      "value": false
    },
    {
      "key": "acomLock",
      "type": "bool",
      "value": false
    },
    {
      "key": "showIndependenceDayCongratulation",
      "type": "bool",
      "value": false
    },
    {
      "key": "isHolidayNewYearEnabled",
      "type": "bool",
      "value": false
    },
    {
      "key": "VMonitoringSchoolPerformanceByRegionPageLock",
      "type": "bool",
      "value": false
    },
    {
      "key": "VReportBySchoolPageLock",
      "type": "bool",
      "value": false
    }
  ]
}
```

</details>

Каждый объект в массиве `data` представляет отдельный флаг:

| Поле | Тип | Описание |
|---|---|---|
| `key` | `string` | Ключ настройки (camelCase) |
| `type` | `string` | Тип значения: `bool`, `text` |
| `value` | `boolean \| string` | Значение настройки |

---

### 5. Получить список связанных детей

**`GET /api/v4/os/system/related-children`**

Используется для аккаунтов родителей. Возвращает список детей, привязанных к аккаунту, с информацией об их учебных группах, школе и подписках.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "children": [
      {
        "studentInfo": {
          "studentName": "ИВАНОВ ИВАН",
          "studentFirstName": "ИВАН",
          "studentUserId": 90000001,
          "studentGroupUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
          "studentUuid": "aaaaaaaa-bbbb-cccc-dddd-ffffffffffff",
          "studentSchoolUuid": "aaaaaaaa-bbbb-cccc-dddd-000000000000",
          "studentGroupUuids": [
            "aaaaaaaa-bbbb-cccc-dddd-111111111111",
            "aaaaaaaa-bbbb-cccc-dddd-222222222222"
          ],
          "groupsByYear": {
            "2024": "ПДД 6А",
            "2025": "ПДД 7В"
          },
          "hasFreedomCard": false,
          "hasKcellPromo": false
        },
        "school": {
          "eduYears": [
            {
              "schoolId": 1000001,
              "eduYear": 2025,
              "isCurrent": true,
              "dateStart": "01.09.2025",
              "dateEnd": "25.05.2026",
              "status": "active",
              "canEditPeriods": false
            }
          ],
          "name": "КГУ «Школа №1»",
          "type": "regular",
          "apiVersion": "v4"
        }
      }
    ]
  }
}
```

</details>

---

### 6. Список внешних привязанных аккаунтов

**`GET /api/v4/os/system/external-account/list`**

Возвращает список внешних провайдеров, доступных для привязки к аккаунту (например, `bieducation`), и текущий статус привязки.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": {
    "availableProviders": ["bieducation"],
    "rows": [
      {
        "provider": "bieducation",
        "externalId": null
      }
    ]
  }
}
```

| Поле | Тип | Описание |
|---|---|---|
| `availableProviders` | `string[]` | Список поддерживаемых внешних провайдеров |
| `rows[].provider` | `string` | Название провайдера |
| `rows[].externalId` | `string \| null` | ID аккаунта у провайдера; `null` — не привязан |

---

### 7. Получить список статических модальных окон

**`GET /api/v2/os/static-modal`**

Возвращает список информационных всплывающих окон, которые должны быть показаны пользователю после входа (например, объявления, уведомления от администрации).

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": []
}
```

> Пустой массив означает, что активных модальных окон для пользователя нет.

---

### 8. Получить баннеры

**`GET /api/v2/os/banner`**

Возвращает список баннеров для отображения на главной странице личного кабинета.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": []
}
```

---

## Расписание

### 9. Получить расписание уроков на неделю

**`GET /api/v4/os/clientoffice/schedule`**

Возвращает расписание уроков на неделю для указанного ученика, разбитое по дням. Для каждого дня указывается список уроков с информацией о предмете, времени, кабинете и учителе.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |
| `studentSchoolUuid` | `string` (UUID) | ✅ | UUID ученика в данной школе |
| `date` | `string` | ✅ | Любая дата из нужной недели в формате `DD.MM.YYYY` |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "days": [
      {
        "date": 1781463600,
        "dateFormat": "15.06.2026",
        "date_label": "15 ИЮНЯ",
        "isCurrent": false,
        "isHoliday": false,
        "schedule": [
          {
            "uuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
            "label": "Алгебра",
            "timeslot": "08:00 - 08:45",
            "cabinet": "204",
            "teacherFio": "Иванова Анна Ивановна",
            "type": "regular",
            "theme": "Квадратные уравнения",
            "homeworkBody": "§10, задачи 1–5"
          }
        ]
      },
      {
        "date": 1781550000,
        "dateFormat": "16.06.2026",
        "date_label": "16 ИЮНЯ",
        "isCurrent": false,
        "isHoliday": false,
        "schedule": []
      }
    ],
    "weeks": {
      "next": "26.06.2026",
      "prev": "12.06.2026",
      "next_time": 1782414000,
      "prev_time": 1781204400
    }
  }
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `days` | `array` | Массив из 7 дней недели |
| `days[].date` | `integer` | Unix-timestamp начала дня |
| `days[].dateFormat` | `string` | Дата в формате `DD.MM.YYYY` |
| `days[].isCurrent` | `boolean` | Является ли день сегодняшним |
| `days[].isHoliday` | `boolean` | Является ли день праздничным/выходным |
| `days[].schedule` | `array` | Список уроков на этот день |
| `weeks.next` / `weeks.prev` | `string` | Дата начала следующей/предыдущей недели |

---

### 10. Получить список групп/классов

**`GET /api/v4/os/clientoffice/group-list`**

Возвращает список всех учебных групп (классов, факультативов, кружков), в которых состоит ученик в данном учебном году.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "value": 930946,
      "label": "7 В класс"
    },
    {
      "value": 1077198,
      "label": "ПДД 7В"
    },
    {
      "value": 1073041,
      "label": "Классный час 7В"
    },
    {
      "value": 1072206,
      "label": "Глобальные компетенции 7В"
    }
  ]
}
```

| Поле | Тип | Описание |
|---|---|---|
| `value` | `integer` | ID группы (используется как `groupId` в других запросах) |
| `label` | `string` | Отображаемое название группы |

---

## Электронный дневник и оценки

### 11. Получить список недель учебного периода

**`GET /api/v4/os/clientoffice/diary/weeks`**

Возвращает навигационную структуру по неделям: текущую неделю, список всех недель учебного года и информацию о периодах (четвертях).

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |
| `period` | `integer` | ❌ | Номер четверти (если не указан — возвращает текущий) |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "week": "20.10.2025",
    "weeks": [
      {
        "label": "Неделя 1",
        "value": "01.09.2025"
      },
      {
        "label": "Неделя 2",
        "value": "08.09.2025"
      },
      {
        "label": "Неделя 7",
        "value": "13.10.2025"
      },
      {
        "label": "Неделя 8",
        "value": "20.10.2025"
      }
    ],
    "periods": [
      {
        "id": 16727725,
        "uuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
        "eduYear": 2025,
        "periodType": "quarter",
        "period": 1,
        "periodStart": "02.09.2025",
        "periodEnd": "24.10.2025",
        "status": "active",
        "recommendedFormula": "2024",
        "isCurrent": false
      }
    ]
  }
}
```

</details>

---

### 12. Получить дневник за неделю

**`GET /api/v4/os/clientoffice/diary`**

Возвращает дневник ученика за указанную неделю: расписание уроков, темы занятий, домашние задания и, если имеются, оценки (баллы).

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |
| `date` | `string` | ✅ | Дата понедельника нужной недели в формате `DD.MM.YYYY` |
| `groupId` | `integer` | ✅ | ID основного класса ученика |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "date": "20 - 26 октябрь 2025",
    "days": [
      {
        "day": "понедельник",
        "date": "20 октября",
        "isHoliday": false,
        "subjects": [
          {
            "scheduleUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
            "label": "Алгебра",
            "homeworkUuid": "aaaaaaaa-bbbb-cccc-dddd-111111111111",
            "type": "regular",
            "sMaxScore": 10,
            "homeworkBody": "повторите основные понятия",
            "homeworkType": "simple",
            "homeworkBooks": [],
            "hasFiles": false,
            "hasBooks": false,
            "timeslot": "08:00 - 08:45",
            "cabinet": null,
            "theme": "Тождественные преобразования выражений",
            "teacherFio": "Иванова Анна Ивановна"
          },
          {
            "scheduleUuid": "aaaaaaaa-bbbb-cccc-dddd-222222222222",
            "label": "Алгебра",
            "homeworkUuid": "aaaaaaaa-bbbb-cccc-dddd-333333333333",
            "type": "soch",
            "sMaxScore": 20,
            "homeworkBody": "№10.1–10.3",
            "homeworkType": "simple",
            "timeslot": "08:50 - 09:35",
            "cabinet": null,
            "theme": "Суммативное оценивание за четверть",
            "teacherFio": "Иванова Анна Ивановна"
          }
        ]
      }
    ]
  }
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `subjects[].type` | `string` | Тип урока: `regular` — обычный, `soch` — суммативная оценка за четверть (СОЧ), `sor` — суммативная оценка за раздел (СОР) |
| `subjects[].sMaxScore` | `integer` | Максимальный балл за работу |
| `subjects[].homeworkBody` | `string` | Текст домашнего задания |
| `subjects[].hasFiles` | `boolean` | Есть ли прикреплённые файлы к ДЗ |
| `subjects[].hasBooks` | `boolean` | Есть ли прикреплённые учебники к ДЗ |

---

### 13. Получить учебные периоды (четверти)

**`GET /api/v4/os/clientoffice/diary/periods`**

Возвращает список учебных периодов (четвертей/семестров) для данного учебного года.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": {
    "periods": [
      {
        "eduYear": 2025,
        "periodType": "quarter",
        "period": 1,
        "periodStart": "02.09.2025",
        "periodEnd": "24.10.2025",
        "isOwner": true,
        "id": 50401,
        "uuid": "50401",
        "isCurrent": false
      },
      {
        "eduYear": 2025,
        "periodType": "quarter",
        "period": 2,
        "periodStart": "03.11.2025",
        "periodEnd": "26.12.2025",
        "isOwner": true,
        "id": 50402,
        "uuid": "50402",
        "isCurrent": false
      },
      {
        "eduYear": 2025,
        "periodType": "quarter",
        "period": 3,
        "periodStart": "08.01.2026",
        "periodEnd": "18.03.2026",
        "isOwner": true,
        "id": 50403,
        "uuid": "50403",
        "isCurrent": false
      },
      {
        "eduYear": 2025,
        "periodType": "quarter",
        "period": 4,
        "periodStart": "30.03.2026",
        "periodEnd": "25.05.2026",
        "isOwner": true,
        "id": 50404,
        "uuid": "50404",
        "isCurrent": false
      }
    ]
  }
}
```

| Поле | Тип | Описание |
|---|---|---|
| `periodType` | `string` | Тип периода: `quarter` — четверть |
| `period` | `integer` | Номер периода (1–4) |
| `isCurrent` | `boolean` | Является ли период текущим |
| `isOwner` | `boolean` | Принадлежит ли период данной школе |

---

### 14. Получить оценки по предметам за период

**`GET /api/v4/os/clientoffice/diary/subjects`**

Возвращает детальный журнал оценок ученика по каждому предмету за указанный учебный период. Включает список всех занятий с датами и выставленными баллами.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |
| `period` | `integer` | ✅ | Номер четверти (1–4) |
| `periodType` | `string` | ✅ | Тип периода: `quarter` |
| `groupId` | `integer` | ✅ | ID класса |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": [
    {
      "eduSubjectUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
      "subjectId": 110,
      "subjectName": "Геометрия",
      "finalScore": "5",
      "periodType": "quarter",
      "periodInfo": {
        "periodUuid": "aaaaaaaa-bbbb-cccc-dddd-111111111111",
        "recommendedFormula": "2024",
        "eduSubjectUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
        "subjectId": 110,
        "scoreType": "mark"
      },
      "scoreType": "mark",
      "schedules": [
        {
          "uuid": "aaaaaaaa-bbbb-cccc-dddd-222222222222",
          "date": "02.09.2025",
          "timeStart": "08:00:00",
          "markMax": 10,
          "type": "regular"
        },
        {
          "uuid": "aaaaaaaa-bbbb-cccc-dddd-333333333333",
          "date": "04.09.2025",
          "timeStart": "08:00:00",
          "markMax": 10,
          "type": "regular"
        }
      ]
    }
  ]
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `subjectName` | `string` | Название предмета |
| `finalScore` | `string \| null` | Итоговая оценка за период |
| `scoreType` | `string` | Система оценивания: `mark` — традиционная |
| `schedules` | `array` | Список занятий с оценками и максимальными баллами |
| `schedules[].markMax` | `integer` | Максимальный балл за занятие |
| `schedules[].type` | `string` | Тип занятия: `regular`, `sor`, `soch` |

---

### 15. Получить итоговые оценки за год

**`GET /api/v4/os/clientoffice/diary/year`**

Возвращает итоговую ведомость ученика за весь учебный год: четвертные и годовые оценки по всем предметам, экзаменационные результаты и сведения о пропущенных занятиях.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "groupName": "7 А",
    "groupId": 930945,
    "rows": [
      {
        "subjectName": "Алгебра",
        "eduSubjectUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
        "scoreType": "mark",
        "periodType": "quarter",
        "attendances": {
          "missingByAnotherReason": 0,
          "missingBySick": 0,
          "missingDue": 0,
          "missingWithoutReason": 3,
          "totalMissCount": 3
        },
        "attestations": {
          "quarter1": "5",
          "quarter2": "5",
          "quarter3": "5",
          "quarter4": "5"
        },
        "yearScores": {
          "studentGroupUuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
          "eduProgramSubjectUuid": "aaaaaaaa-bbbb-cccc-dddd-111111111111",
          "recommendedYearScore": "5",
          "yearScore": "5",
          "examScore": null,
          "recommendedFinalScore": "5",
          "finalScore": "5",
          "uuid": "aaaaaaaa-bbbb-cccc-dddd-222222222222"
        }
      }
    ]
  }
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `attendances.missingBySick` | `integer` | Пропусков по болезни |
| `attendances.missingWithoutReason` | `integer` | Пропусков без причины |
| `attendances.totalMissCount` | `integer` | Всего пропущено занятий |
| `attestations.quarterN` | `string \| null` | Оценка за N-ю четверть |
| `yearScores.yearScore` | `string \| null` | Годовая оценка |
| `yearScores.examScore` | `string \| null` | Экзаменационная оценка |
| `yearScores.finalScore` | `string \| null` | Итоговая оценка |

---

## Домашние задания

### 16. Получить список домашних заданий за месяц

**`GET /api/v4/os/clientoffice/homeworks/monthly/list`**

Возвращает сгруппированный по месяцам список домашних заданий для указанной учебной группы.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |
| `studentGroupUuid` | `string` (UUID) | ✅ | UUID основной учебной группы ученика |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "month": "Сентябрь 2025",
      "homeworks": [
        {
          "uuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
          "subjectName": "Алгебра",
          "date": "05.09.2025",
          "body": "§1, задачи 1–10",
          "type": "simple",
          "hasFiles": false
        }
      ]
    }
  ]
}
```

> **Примечание:** Если ДЗ не назначены, возвращается `{"data": []}`.

---

## Чаты

### 17. Получить список доступных собеседников

**`GET /api/v4/os/clientoffice/chat/personal-chats-data`**

Возвращает список пользователей (учителей, родителей), с которыми ученик может начать личную переписку.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "userId": 90000002,
      "name": "Иванова Анна Ивановна"
    },
    {
      "userId": 90000003,
      "name": "Петров Сергей Николаевич"
    }
  ]
}
```

| Поле | Тип | Описание |
|---|---|---|
| `userId` | `integer` | ID пользователя в системе |
| `name` | `string` | ФИО пользователя |

---

### 18. Получить активные личные чаты

**`GET /api/v4/os/clientoffice/chat/personal-chats`**

Возвращает список активных личных чатов текущего пользователя с историей последних сообщений.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "chatId": "chat_90000001_90000002",
      "userId": 90000002,
      "name": "Иванова Анна Ивановна",
      "lastMessage": {
        "text": "Добрый день!",
        "createdAt": "2026-06-18T15:30:00Z"
      },
      "unreadCount": 1
    }
  ]
}
```

> **Примечание:** Если активных чатов нет, возвращается `{"data": []}`.

---

## Профиль и безопасность

### 19. Проверить необходимость смены пароля

**`GET /api/v2/os/profile/need-pass-change`**

Проверяет, был ли пароль пользователя изменён с дефолтного. Если `defaultPass: 1` — рекомендуется показать пользователю форму смены пароля.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": {
    "defaultPass": 1
  }
}
```

| Поле | Значение | Описание |
|---|---|---|
| `defaultPass` | `1` | Пользователь использует стандартный пароль — необходима смена |
| `defaultPass` | `0` | Пароль уже изменён пользователем |

---

### 20. Получить список родителей

**`GET /api/v4/os/clientoffice/parent-list`**

Возвращает список родителей/законных представителей, привязанных к аккаунту ученика.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": [
    {
      "name": "Иванова Екатерина Ивановна",
      "phone": "+7 (700) 000-00-00",
      "date": "26.09.2024"
    }
  ]
}
```

| Поле | Тип | Описание |
|---|---|---|
| `name` | `string` | ФИО родителя |
| `phone` | `string` | Номер телефона |
| `date` | `string` | Дата привязки аккаунта родителя в формате `DD.MM.YYYY` |

---

### 21. Получить активные сессии

**`GET /api/v2/os/active-sessions`**

Возвращает список текущих активных сессий пользователя с информацией об устройстве, браузере и IP-адресе.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

```json
{
  "data": {
    "currentSessionId": 203097036,
    "rows": [
      {
        "id": {
          "label": 203097036
        },
        "time": {
          "label": "01:14",
          "additional": "19.06.2026"
        },
        "ip": {
          "label": "192.0.2.1"
        },
        "device": {
          "type": "Мобильный телефон",
          "os": "Android 15"
        },
        "info": {
          "label": "Chrome 149"
        },
        "exitTime": null
      }
    ]
  }
}
```

| Поле | Тип | Описание |
|---|---|---|
| `currentSessionId` | `integer` | ID текущей сессии пользователя |
| `rows[].time` | `object` | Время входа: `label` — время, `additional` — дата |
| `rows[].ip.label` | `string` | IP-адрес входа |
| `rows[].device` | `object` | Тип устройства и ОС |
| `rows[].info.label` | `string` | Название браузера |
| `rows[].exitTime` | `object \| null` | Время выхода (`null` если сессия активна) |

---

### 22. Получить завершённые сессии

**`GET /api/v2/os/ended-sessions`**

Возвращает историю завершённых сессий пользователя с пагинацией.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `schoolId` | `integer` | ✅ | ID школы |
| `eduYear` | `integer` | ✅ | Учебный год |

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "count": 166,
    "rows": [
      {
        "id": {
          "label": 196786826
        },
        "time": {
          "label": "09:09",
          "additional": "25.05.2026"
        },
        "ip": {
          "label": "192.0.2.10"
        },
        "device": {
          "type": "Мобильный телефон",
          "os": "Android"
        },
        "info": {
          "label": "Android Browser"
        },
        "exitTime": {
          "label": "15:54",
          "additional": "29.05.2026"
        }
      }
    ]
  }
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `count` | `integer` | Общее количество завершённых сессий |
| `rows[].exitTime` | `object` | Время выхода из системы |

---

## Новости (гостевой доступ)

### 23. Получить публичные статьи / новости

**`GET /api/v2/os/guest/article`**

Публичный эндпоинт (не требует авторизации). Возвращает список новостных статей, опубликованных на платформе. Используется для отображения на странице входа.

#### Query-параметры

| Параметр | Тип | Обязателен | Описание |
|---|---|---|---|
| `eduYear` | `integer` | ✅ | Учебный год |

> **Авторизация не требуется.** Заголовок `Authorization` не нужен.

#### Ответ — `200 OK`

<details>
<summary>Посмотреть пример ответа</summary>

```json
{
  "data": {
    "current_page": 1,
    "data": [
      {
        "id": 7320,
        "article_id": 3520,
        "status": 1,
        "title": "Школы нового формата – это не просто здания, а комфортная и безопасная среда для обучения",
        "preview": "https://bilimland.kz/uploads/articles/3520/preview_ru_3520_xxx.jpeg",
        "published_at": "2025-10-24 11:52:00",
        "slug": "skoly-novogo-formata-eto-ne-prosto-zdaniya",
        "origin_link": "https://www.gov.kz/memleket/entities/edu/press/news/details/1092571?lang=ru",
        "origin": {
          "link": "https://www.gov.kz/memleket/entities/edu/press/news/details/1092571?lang=ru",
          "label": "МОН РК"
        }
      }
    ],
    "from": 1,
    "last_page": 5,
    "per_page": 10,
    "to": 10,
    "total": 48
  }
}
```

</details>

| Поле | Тип | Описание |
|---|---|---|
| `current_page` | `integer` | Текущая страница пагинации |
| `last_page` | `integer` | Последняя страница |
| `per_page` | `integer` | Количество статей на странице |
| `total` | `integer` | Общее количество статей |
| `data[].title` | `string` | Заголовок статьи |
| `data[].preview` | `string` | URL превью-изображения |
| `data[].slug` | `string` | URL-slug статьи |
| `data[].origin.label` | `string` | Источник статьи (например, `МОН РК`) |

---

## Коды ошибок

| HTTP-статус | Описание |
|---|---|
| `200 OK` | Запрос выполнен успешно |
| `401 Unauthorized` | Токен отсутствует, истёк или недействителен |
| `403 Forbidden` | Недостаточно прав для выполнения операции |
| `404 Not Found` | Запрашиваемый ресурс не найден |
| `500 Internal Server Error` | Внутренняя ошибка сервера (например, временная недоступность БД) |

> **Пример ответа при ошибке авторизации:**
> ```json
> {
>   "message": "Unauthenticated."
> }
> ```

---

## Примечания

- **Формат дат в параметрах:** `DD.MM.YYYY` (например, `19.06.2026`).
- **Формат дат в ответах:** чаще всего `DD.MM.YYYY`, иногда ISO 8601 (`YYYY-MM-DDTHH:mm:ssZ`).
- **Учебный год:** значение `2025` означает учебный год `2025–2026`. Запросы с неактивным учебным годом вернут данные из архива.
- **Параметр `eduYear` в query-строке** и **в теле запроса** (при POST) — один и тот же параметр, может присутствовать в обоих местах одновременно.
- **Сервер:** Kubernetes-кластер (`bilimclass-backend-*`), ответы сжимаются через `gzip`.
- **Задержки:** среднее время ответа сервера — 290–400 мс (данные из `Server-Timing`).
