# CORS (Cross-Origin Resource Sharing)

## Что такое CORS?

**CORS (Cross-Origin Resource Sharing)** — это механизм безопасности в веб-браузерах, который позволяет браузеру отправлять запросы к другому источнику (ограничивает) запросы к ресурсам с других доменов, протоколов или портов.

По умолчанию браузеры блокируют JavaScript-запросы к другому источнику (origin), чем тот, с которого загружена страница. Это называется **Same-Origin Policy** (политика одного источника).

## Из чего состоит Origin (источник)?

Origin = схема + домен + порт

```
https://example.com:443
↑        ↑            ↑
схема    домен       порт
```

Примеры:
- `https://site.com` и `https://api.site.com` — **разные** источники (разные домены)
- `http://localhost:3000` и `http://localhost:8080` — **разные** источники (разные порты)
- `https://site.com` и `http://site.com` — **разные** источники (разные протоколы)

## Зачем нужен CORS?

Без CORS любой сайт мог бы делать запросы к любому другому сайту от имени пользователя. Это создало бы серьёзные уязвимости:

- Кража cookies и данных аутентификации
- CSRF-атаки (подделка межсайтовых запросов)
- Доступ к приватным данным пользователя

## Как работает CORS?

### Простой запрос (simple request)

Для простых запросов (GET, POST с определёнными типами контента) браузер отправляет запрос с заголовком:

```
Origin: https://example.com
```

Сервер в ответ может отправить заголовки:

```
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
```

Если источник разрешён — запрос выполняется. Если нет — браузер блокирует ответ.

### Preflight-запрос (предварительный запрос)

Для "сложных" запросов (PUT, DELETE, кастомные заголовки и т.д.) браузер сначала отправляет **OPTIONS**-запрос:

```
OPTIONS /api/data HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
```

Сервер должен ответить разрешениями, и только потом браузер отправит основной запрос.

## Основные заголовки CORS

| Заголовок | Описание |
|-----------|----------|
| `Access-Control-Allow-Origin` | Разрешённый источник (`*` для любого) |
| `Access-Control-Allow-Methods` | Разрешённые HTTP-методы |
| `Access-Control-Allow-Headers` | Разрешённые заголовки запроса |
| `Access-Control-Allow-Credentials` | Разрешение передачи cookies (`true`) |
| `Access-Control-Max-Age` | Сколько секунд кешировать preflight-ответ |

## Ограничения CORS

1. **Браузерная защита** — CORS работает только в браузерах. Серверные запросы (curl, Node.js, Python) не ограничены CORS.

2. **Access-Control-Allow-Origin с credentials** — если используешь `Access-Control-Allow-Credentials: true`, нельзя использовать `*` в `Access-Control-Allow-Origin`. Нужно указывать конкретный источник.

3. **Preflight кеширование** — можно кешировать preflight-ответы с помощью `Access-Control-Max-Age`, но это не поддерживается везде одинаково.

4. **Сложность с несколькими источниками** — если нужно разрешить много источников, сервер должен динамически формировать заголовок на основе `Origin` из запроса.

## Пример настройки CORS

### Node.js/Express

```javascript
const express = require('express');
const cors = require('cors');

const app = express();

// Разрешить все источники (небезопасно для API с credentials)
app.use(cors());

// Или более точно:
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}));
```

### Nginx

```nginx
location /api/ {
  add_header Access-Control-Allow-Origin "https://example.com";
  add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
  add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
  
  if ($request_method = OPTIONS) {
    return 204;
  }
}
```

## Как отладить CORS?

1. **Консоль браузера** — обычно там видно ошибку "Access to fetch at ... blocked by CORS policy"
2. **Вкладка Network** — смотри заголовки запроса и ответа
3. **Расширения** — временно отключить CORS для разработки (только для dev!)
4. **Серверная сторона** — проверить, что сервер отправляет правильные заголовки

## Итог

`CORS` — это механизм, который позволяет серверу контролировать, какие веб-страницы могут получать доступ к его ресурсам.

Это важный элемент безопасности, но он может создавать сложности при разработке, особенно при работе с API из разных источников.
