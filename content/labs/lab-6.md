## 1. Тема, мета, посилання

### 1.1 Тема

«Документування API за допомогою Swagger. Деплой Node.js-додатку. Підсумковий проєкт: REST API з базою даних та Swagger документацією».

### 1.2 Мета

Інтегрувати Swagger/OpenAPI-документацію в існуючий backend Scoring figure skating app, описати основні endpoint-и REST API, перевірити API через Swagger UI та підготувати застосунок до production-запуску і деплою.

### 1.3 Посилання

- Репозиторій власного веб-застосунку (GitHub): [посилання](https://github.com/hannoveran/scoring-figure-skating-app.git)
- Власний веб-застосунок (Жива сторінка): [посилання](https://hannoveran.github.io/scoring-figure-skating-app/)
- Репозиторій звітного HTML-документа (GitHub): [посилання](https://github.com/hannoveran/IC-31_appRECORD-MamchukAnna-FIOT-2026.git)
- Звітний HTML-документ (Жива сторінка): [посилання](https://hannoveran.github.io/IC-31_appRECORD-MamchukAnna-FIOT-2026/)

---

## 2. Короткі теоретичні відомості

### 2.1 REST API

REST API — це архітектурний підхід до організації взаємодії між клієнтською та серверною частинами веб-застосунку через HTTP-протокол. У розробленому проєкті Figure Skating Scoring System REST API використовується для роботи з автентифікацією користувачів, керуванням змаганнями, отриманням результатів, завантаженням файлів і моніторингом стану сервера.

Основні HTTP-методи, використані у проєкті:
- `GET` — отримання даних із сервера;
- `POST` — створення нових записів та авторизація;
- `PUT` — оновлення існуючих даних;
- `PATCH` — часткове оновлення профілю користувача;
- `DELETE` — видалення записів.

Для реалізації REST API використано Node.js та Express.js.

### 2.2 Swagger та OpenAPI

OpenAPI Specification — це стандарт документування REST API, який дозволяє описувати маршрути, параметри запитів, структури даних, відповіді сервера та механізми авторизації. На основі OpenAPI-документа Swagger UI автоматично створює інтерактивний web-інтерфейс для тестування API.

У проєкті Swagger використовується для:
- документування endpoint-ів;
- тестування API без використання сторонніх програм;
- перевірки роботи JWT-автентифікації;
- перегляду структури запитів і відповідей.

Для інтеграції Swagger використано пакети:
- `swagger-jsdoc` — генерація OpenAPI-документації з JSDoc-коментарів;
- `swagger-ui-express` — підключення Swagger UI до Express-застосунку.

Swagger UI у проєкті доступний через маршрут:

```
http://localhost:3001/api-docs
```

### 2.3 Підключення бази даних через Sequelize

Sequelize — це ORM-бібліотека для Node.js, яка забезпечує взаємодію між JavaScript-застосунком і реляційною базою даних. У проєкті Sequelize використовується для виконання CRUD-операцій, створення моделей і роботи із SQL Server.

У застосунку реалізовано моделі:
- `User`;
- `Competition`;
- `Program`;
- `Score`;
- `Skater`.

Для підключення використовується СУБД Microsoft SQL Server (MSSQL) та діалект `mssql`.

Sequelize дозволяє:
- виконувати SQL-операції через JavaScript;
- працювати з асоціаціями між таблицями;
- реалізовувати CRUD-операції;
- спрощувати підтримку backend-коду.

### 2.4 JWT-автентифікація

JWT (JSON Web Token) — це механізм автентифікації, який використовується для захисту API. Після успішного входу користувач отримує токен доступу, який надалі передається у заголовках HTTP-запитів.

У проєкті JWT використовується для:
- авторизації користувачів;
- захисту приватних endpoint-ів;
- перевірки ролей користувачів;
- роботи механізму refresh token.

Для роботи з JWT використано бібліотеку `jsonwebtoken`.

---

## 3. Реалізований функціонал Lab 6

### 3.1 Основні сценарії

У межах лабораторної роботи реалізовано REST API для веб-застосунку Figure Skating Scoring System. Backend-частина побудована на Node.js та Express.js із використанням Sequelize ORM і Microsoft SQL Server.

У проєкті реалізовано:
- REST API для роботи із системою оцінювання фігурного катання;
- CRUD-операції для змагань;
- JWT-автентифікацію користувачів;
- Swagger UI для документування та тестування API;
- endpoint-и для авторизації, результатів, моніторингу та завантаження файлів;
- middleware для логування, обробки помилок і вимірювання часу відповіді;
- Redis-кешування;
- rate limiting та Helmet для захисту API;
- Swagger-документацію для основних маршрутів API.

### 3.2 Адаптація під існуючий проєкт

У межах лабораторної роботи приклади реалізації наведено для Node.js та MySQL. У поточному проєкті серверна частина вже була реалізована на Express.js із використанням Sequelize ORM та Microsoft SQL Server (MSSQL).

Оскільки проєкт уже містив REST API, CRUD-операції, JWT-автентифікацію та підключення до бази даних, Swagger/OpenAPI було інтегровано без зміни архітектури серверної частини.

Для документування API використано:
- `swagger-jsdoc` — генерація OpenAPI-документації;
- `swagger-ui-express` — підключення Swagger UI до Express-застосунку.

---

## 4. Реалізація Swagger/OpenAPI

### 4.1 Конфігурація OpenAPI

Swagger-конфігурацію винесено в окремий файл `server/config/swagger.js`. У ньому описано основну інформацію про API, сервер, схеми авторизації та моделі даних.

```
const swaggerJSDoc = require('swagger-jsdoc');

const swaggerSpec = swaggerJSDoc({
  definition: {
    openapi: '3.0.0',

    info: {
      title: 'Figure Skating Scoring API',
      version: '1.0.0',
      description: 'REST API for figure skating scoring system',
    },

    servers: [
      {
        url: 'http://localhost:3001',
      },
    ],
  },
});
```

У документації описано основні маршрути:
- `/api/auth/register`
- `/api/auth/login`
- `/api/auth/me`
- `/api/competitions`
- `/api/competitions/{id}`
- `/api/results`
- `/api/upload`
- `/api/status`

### 4.2 Підключення Swagger UI до Express

Swagger UI підключено у файлі `server/app.js` за допомогою `swagger-ui-express`.

```
const swaggerUi = require('swagger-ui-express');
const swaggerSpec = require('./config/swagger');

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

Після запуску сервера документація доступна за адресою:
```
http://localhost:3001/api-docs
```

Swagger UI дозволяє:
- переглядати всі endpoint-и;
- тестувати API без Postman;
- надсилати HTTP-запити;
- перевіряти JWT-авторизацію;
- переглядати структуру request/response.

### 4.3 Опис моделей даних

У Swagger/OpenAPI описано основні схеми даних, які використовуються API:
- `User` — користувач системи;
- `Competition` — змагання;
- `Error` — стандартна помилка API;
- `Success` — успішна відповідь сервера.

Приклад опису моделі `Competition`:

```
Competition: {
  type: 'object',
  properties: {
    id: { type: 'integer' },
    name: { type: 'string' },
    date: { type: 'string' },
    location: { type: 'string' },
    category: { type: 'string' },
    segment: { type: 'string' },
    status: { type: 'string' },
  },
}
```

### 4.4 JWT-автентифікація у Swagger

Для тестування захищених маршрутів у Swagger реалізовано Bearer JWT authorization.

У конфігурації Swagger додано схему `bearerAuth`:

```
securitySchemes: {
  bearerAuth: {
    type: 'http',
    scheme: 'bearer',
    bearerFormat: 'JWT',
  },
}
```

Після авторизації користувач може тестувати захищені endpoint-и безпосередньо через Swagger UI за допомогою кнопки `Authorize`.

Для отримання токена використовується endpoint: `POST /api/auth/login`

Отриманий `accessToken` передається у форматі: `Bearer YOUR_TOKEN`

Це дозволяє перевіряти роботу захищених маршрутів API без використання сторонніх інструментів.

---

## 5. Перевірка API через Swagger UI

Після запуску backend-застосунку документація Swagger UI доступна за адресою: `http://localhost:3001/api-docs`

Swagger UI автоматично відображає список основних груп endpoint-ів:
- Auth;
- Competitions;
- Results;
- Upload;
- Status.

Через Swagger UI можна тестувати API без використання Postman або інших сторонніх інструментів.

Було перевірено такі endpoint-и:
- `POST /api/auth/register` — реєстрація користувача;
- `POST /api/auth/login` — вхід у систему;
- `GET /api/auth/me` — отримання даних поточного користувача;
- `GET /api/competitions` — отримання списку змагань;
- `POST /api/competitions` — створення нового змагання;
- `PUT /api/competitions/{id}` — оновлення змагання;
- `DELETE /api/competitions/{id}` — видалення змагання;
- `GET /api/results` — отримання результатів;
- `POST /api/upload` — завантаження файлів;
- `GET /api/status` — моніторинг стану сервера.

Після успішної авторизації Swagger UI дозволяє виконувати запити до захищених маршрутів за допомогою Bearer JWT token.

---

## 6. Тестування REST API

Для перевірки роботи REST API використовувався Swagger UI. Через інтерфейс Swagger було протестовано основні CRUD-операції та механізм JWT-автентифікації.

Під час тестування перевірено:
- коректність HTTP-відповідей;
- створення записів у базі даних;
- оновлення та видалення даних;
- обробку помилок;
- роботу middleware;
- JWT-захист endpoint-ів;
- завантаження файлів через Multer.

Для тестування авторизації використовувався endpoint: `POST /api/auth/login`

Після успішного login Swagger повертає `accessToken`, який використовується для авторизації захищених маршрутів через кнопку `Authorize`.

---

## 7. Запуск серверного застосунку

### 7.1 Запуск backend-сервера

Для запуску серверної частини застосунку використовувалась команда:

```
npm run dev
```

Після запуску backend API стає доступним за адресою:

```
http://localhost:3001
```

### 7.2 Змінні середовища

Для роботи застосунку використовуються змінні середовища, які зберігаються у файлі `.env`.

Основні змінні:
- `JWT_SECRET` — секретний ключ JWT;
- `PORT` — порт сервера;
- параметри підключення до MSSQL;
- параметри Redis.

### 7.3 Підключення бази даних

У проєкті використовується Microsoft SQL Server (MSSQL) через Sequelize ORM.

Підключення до бази даних реалізовано у файлі:
```
server/config/db.js
```

Sequelize забезпечує:
- CRUD-операції;
- SQL-запити через JavaScript;
- асоціації між моделями;
- взаємодію з таблицями бази даних.

---

## 8. Результати виконання

![Успішний запуск backend-застосунку Figure Skating Scoring API](/assets/labs/lab-6/api_dev_server_running.png)
**Рис. 1 – Успішний запуск backend-застосунку Figure Skating Scoring API через `npm run dev`.**

![Swagger UI Figure Skating Scoring API](/assets/labs/lab-6/swagger_opened.png)
**Рис. 2 – Swagger UI за адресою `http://localhost:3001/api-docs` зі списком endpoint-ів API.**

![Swagger login endpoint](/assets/labs/lab-6/swagger_login_success.png)
**Рис. 3 – Документація endpoint-а `POST /api/auth/login` у Swagger UI.**

![JWT Bearer authorization у Swagger UI](/assets/labs/lab-6/swagger_bearer.png)
**Рис. 4 – Авторизація через Bearer JWT token у Swagger UI.**

![Створення нового змагання через Swagger UI](/assets/labs/lab-6/swagger_post_competitions.png)
**Рис. 5 – Виконання запиту `POST /api/competitions` через Swagger UI.**

![Отримання списку змагань через REST API](/assets/labs/lab-6/swagger_get_competitions.png)
**Рис. 6 – Успішне виконання `GET /api/competitions` та отримання даних із бази MSSQL.**

![Перевірка endpoint-а моніторингу стану сервера](/assets/labs/lab-6/swagger_status.png)
**Рис. 7 – Перевірка endpoint-а `GET /api/status` із інформацією про uptime та використання пам’яті сервера.**

---

## 9. Висновки

У межах лабораторної роботи до веб-застосунку Figure Skating Scoring System інтегровано Swagger/OpenAPI-документацію для REST API. Для backend-системи створено OpenAPI-документ із описом основних endpoint-ів, моделей даних, параметрів запитів, відповідей сервера та JWT Bearer авторизації. Swagger UI підключено до Express-застосунку та зроблено доступним за маршрутом `/api-docs`.

Документація охоплює основні можливості backend-проєкту: авторизацію користувачів, CRUD-операції для змагань, отримання результатів, upload файлів і моніторинг стану сервера. За допомогою Swagger UI виконано тестування REST API, перевірено роботу JWT-автентифікації та коректність HTTP-відповідей.

У процесі виконання лабораторної роботи було використано Node.js, Express.js, Sequelize ORM, Microsoft SQL Server (MSSQL), JWT, Redis і Swagger/OpenAPI. Також у проєкті реалізовано логування, обробку помилок, кешування, middleware безпеки, rate limiting і файлове завантаження.

Отриманий результат є завершеним backend-рішенням для системи оцінювання виступів із фігурного катання з REST API, базою даних і повноцінною Swagger-документацією.

---

## 10. Перелік використаних джерел

1. Документація OpenAPI Specification.
2. Документація Swagger UI.
3. Документація Express.js.
4. Документація Sequelize ORM.
5. Документація Microsoft SQL Server.
6. Документація swagger-jsdoc.
7. Документація swagger-ui-express.
8. Документація JWT (jsonwebtoken).
9. Документація Node.js.