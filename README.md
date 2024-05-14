# Задание для Backend (PHP) разработчика

## Задание

Необходимо реализовать RESTful API для редактирования любого json-документа методом PATCH.
Пользователь должен иметь возможность создать пустой черновик документа.
Пока документ находится в статусе черновик, его можно редактировать сколько угодно раз.
Черновик документа можно опубликовать.
После публикации документ больше редактировать нельзя.
Реализовать все **GET**/**POST**/**DELETE**/**PUT**/**PATCH** запросы.
Закрытые запросы АПИ должны быть доступны только под авторизацией через токен, передаваемый в заголовке.
Для получения/обновления/удаления токена создан отдельный блок роутов для каждого действия.
Локальная swagger документация с примерами запросов и ответов. Собираться должна автоматически на основании аннотаций в
коде.

## Требования

|                  |         |
|------------------|---------|
| Фреймворк        | Bitrix  |
| Сторонние пакеты | любые   |
| PHP              | >=8.1   |
| DB               | MySQL 8 |
| Авторизация      | OAuth 2 |

1. Разместить код на любом доступном git-репозитории либо передать архивом.
2. Соблюдать единый code-style на протяжении всего проекта (следовать PSR)
3. Обязательна документация для каждого метода, класса и поля. Указание типов обязательно.

## API

Реализовать как минимум следующие роуты:

- `POST /api/v1/document/` - создаем черновик документа
- `GET /api/v1/document/{uuid}` - получить документ по uuid
- `PATCH /api/v1/document/{uuid}` - редактировать документ
- `PUT /api/v1/document/{uuid}` - обновить документ
- `DELETE /api/v1/document/{uuid}` - удалить документ
- `POST /api/v1/document/{uuid}/publish` - опубликовать документ
- `GET /api/v1/document/?page=1&perPage=20` - получить список документов с пагинацией, сортировка в последние созданные
  сверху.

Дополнительные условия:

- Если документ не найден, то в ответе возвращается статус 404.
- При попытке редактирования документа, который уже опубликован, должен возвращаться статус 400.
- Попытка опубликовать уже опубликованный документ возвращает статус 200.
- Все запросы на конкретный документ возвращают этот документ. [JsonSchema ответа с документом](document-response.json).
- Список документов возвращается в виде массива документов и значений
  пагинации. [JsonSchema списка документов](document-list-response.json).
- Запрос `PATCH` отправляется с телом json в соответствии с иерархией документа, все поля, кроме `payload` игнорируются.
  Если `payload` не передан, то в ответ статус 400.

### Объект документа

```json
{
  "uuid": "21648bea-a134-4767-9e5b-2519669a6a92",
  "status": "draft|published",
  "payload": "Object",
  "dateCreate": 1715719436,
  "dateUpdate": 1715719436
}
```

[JsonSchema для документа](document.json)

## Патч документа

Патч проводится согласно [RFC-7396](https://tools.ietf.org/html/rfc7396).

## Пример работы

### 1. Пользователь делает запрос на создание документа

Запрос:

```http
POST /api/v1/document HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "uuid": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {},
        "dateCreate": 1715719436,
        "dateUpdate": 1715719436
    }
}
```

### 2. Пользователь редактирует документ первый раз

Запрос:

```http
PATCH /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784 HTTP/1.1
accept: application/json
content-type: application/json

{
    "document": {
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "quick",
                "color": "brown"
            },
            "actions": [
                {
                    "action": "jump over",
                    "actor": "lazy dog"
                }
            ]
        }
    }
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "uuid": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "quick",
                "color": "brown"
            },
            "actions": [
                {
                    "action": "jump over",
                    "actor": "lazy dog"
                }
            ]
        },
        "dateCreate": 1715719436,
        "dateUpdate": 1715719436
    }
}
```

### 3. Пользователь редактирует документ

Запрос:

```http
PATCH /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784 HTTP/1.1
accept: application/json
content-type: application/json

{
    "document": {
        "payload": {
            "meta": {
                "type": "cunning",
                "color": null
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        }
    }
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "uuid": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "draft",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "cunning",
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        },
        "dateCreate": 1715719436,
        "dateUpdate": 1715719436
    }
}
```

### 4. Пользователь публикует документ

Запрос:

```http
POST /api/v1/document/718ce61b-a669-45a6-8f31-32ba41f94784/publish HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": {
        "uuid": "718ce61b-a669-45a6-8f31-32ba41f94784",
        "status": "published",
        "payload": {
            "actor": "The fox",
            "meta": {
                "type": "cunning",
            },
            "actions": [
                {
                    "action": "eat",
                    "actor": "blob"
                },
                {
                    "action": "run away"
                }
            ]
        },
        "dateCreate": 1715719436,
        "dateUpdate": 1715719436
    }
}
```

### 5. Пользователь получает запись в списке

Запрос:

```http
GET /api/v1/document/?page=1 HTTP/1.1
accept: application/json
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "document": [
        {
            "uuid": "718ce61b-a669-45a6-8f31-32ba41f94784",
            "status": "published",
            "payload": {
                "actor": "The fox",
                "meta": {
                    "type": "cunning",
                },
                "actions": [
                    {
                        "action": "eat",
                        "actor": "blob"
                    },
                    {
                        "action": "run away"
                    }
                ]
            },
            "dateCreate": 1715719436,
            "dateUpdate": 1715719436
        }
    ],
    "pagination": {
        "page": 1,
        "perPage": 20,
        "total": 1
    }
}
```

### Аутентификация/авторизация

#### 1. Авторизация по логину и паролю `POST /api/v1/auth/login`
Запрос:

```http
POST /api/v1/auth/login HTTP/1.1
accept: application/json
content-type: application/json

{
    "login": "login",
    "password": "password"
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "token": "f52ad2da-374632d3-9429ff85-872baa46",
    "expire": 1668163429,
    "refresh-token": "31022be4-85d420d6-850dfb89-0349d90c",
    "refresh-expire": 1668767929
}
```

#### 2. Обновление токена `POST /api/v1/auth/update`
Запрос:

```http
POST /api/v1/auth/update HTTP/1.1
accept: application/json
content-type: application/json

{
    "refresh_token": "refresh_token"
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "token": "f52ad2da-374632d3-9429ff85-872baa46",
    "expire": 1668163429,
    "refresh-token": "31022be4-85d420d6-850dfb89-0349d90c",
    "refresh-expire": 1668767929
}
```
#### 3. Удаление токена `POST /api/v1/auth/logout`
Запрос:

```http
POST /api/v1/auth/logout HTTP/1.1
accept: application/json
content-type: application/json

{
    "token": "token"
}
```

Ответ:

```http
HTTP/1.1 200 OK
content-type: application/json

{
    "token": "f52ad2da-374632d3-9429ff85-872baa46",
    "expire": 1668163429,
    "refresh-token": "31022be4-85d420d6-850dfb89-0349d90c",
    "refresh-expire": 1668767929
}
```

Требования:

1. Каждый раз когда пользователь авторизуется - он получает новый токен.
2. Если пользователь шлет любой запрос с несуществующим токеном или с просроченным токеном - то должен вернуться ответ с
   кодом 401.
3. Редактировать и опубликовать документ может только пользователь создавший его.
4. Пользователь в списке документов видит только свои неопубликованные документы и опубликованные документы других
   пользователей, но не видит неопубликованные документы других.
5. Пользователь при попытке обратиться к чужому, неопубликованному документу получает 403.
6. Время действия токена - 1 час.
