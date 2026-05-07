## 1. Тема, мета, посилання

### 1.1 Тема
СТВОРЕННЯ БАЗИ ДАНИХ У MYSQL. ПІДКЛЮЧЕННЯ NODE.JS ДО MYSQL. РОБОТА З ORM SEQUELIZE 

### 1.2 Мета
Побудувати реляційну базу даних для власного web-застосунку Scoring figure skating app, підключити backend на Node.js до MYSQL, реалізувати роботу з даними через ORM SEQUELIZE, виконати CRUD-операції для основних сутностей та окремо показати виконання прямих SQL-запитів із серверної частини.

### 1.3 Посилання
- Репозиторій власного веб-застосунку (GitHub): [посилання](https://github.com/hannoveran/scoring-figure-skating-app.git)
- Власний веб-застосунок (Жива сторінка): [посилання](https://hannoveran.github.io/scoring-figure-skating-app/)
- Репозиторій звітного HTML-документа (GitHub): [посилання](https://github.com/hannoveran/IC-31_appRECORD-MamchukAnna-FIOT-2026.git)
- Звітний HTML-документ (Жива сторінка): [посилання](https://hannoveran.github.io/IC-31_appRECORD-MamchukAnna-FIOT-2026/)

---

## 2. Короткі теоретичні відомості

### 2.1 Microsoft SQL Server
У даній роботі використовується Microsoft SQL Server як реляційна система керування базою даних. Вона забезпечує зберігання структурованих даних про змагання, спортсменів, програми та оцінки суддів. Використання зовнішніх ключів дозволяє підтримувати зв’язки між таблицями (наприклад, competition → program → score) та забезпечує цілісність даних у системі оцінювання фігурного катання.

### 2.2 Node.js та Express
Серверна частина реалізована на Node.js із використанням фреймворку Express. Він забезпечує побудову REST API для роботи з основними сутностями системи: competitions, programs, skaters та scores. Express дозволяє організувати маршрутизацію, обробку HTTP-запитів та middleware-логіку (наприклад, CORS і парсинг JSON), що є достатнім для реалізації серверної частини навчального проєкту.

### 2.3 Sequelize ORM
У проєкті використовується Sequelize як ORM (Object-Relational Mapping) для взаємодії з базою даних Microsoft SQL Server. Він дозволяє описувати моделі таблиць у вигляді JavaScript-об’єктів і виконувати операції CRUD через методи findAll, create, update, destroy без написання SQL-запитів вручну. Це спрощує роботу з даними та зменшує кількість низькорівневого коду.

### 2.4 Прямі SQL-запити (MSSQL)
Окрім ORM, у проєкті використовується можливість виконання прямих SQL-запитів через Sequelize або драйвер tedious. Це застосовується для складних операцій, зокрема каскадного видалення зв’язаних записів (program, score), де використання чистого SQL є більш гнучким. Такий підхід демонструє поєднання ORM-рівня та прямої роботи з SQL у межах однієї системи.

---

## 3. Проєктування бази даних

### 3.1 Предметна область
У якості предметної області обрано систему оцінювання змагань з фігурного катання. Система дозволяє організовувати змагання, реєструвати спортсменів, формувати програми виступів та фіксувати оцінки суддів. Така структура забезпечує можливість моделювання реального змагального процесу, включаючи етапи виступів і суддівського оцінювання.

### 3.2 Основні сутності
У лабораторній версії проєкту використано такі основні сутності:
- `User` — користувач системи (суддя, адміністратор або глядач);
- `Competition` — змагання з базовою інформацією (назва, дата, локація);
- `Skater` — спортсмен, який бере участь у змаганнях;
- `Program` — програма виступу (short program або free skate);
- `Score` — оцінка, виставлена суддею за конкретну програму.

### 3.3 Зв’язки між таблицями
У моделі реалізовано кілька зв’язків типу One-to-Many:
- один користувач може створювати багато змагань (`User -> Competition`);
- одне змагання містить багато програм (`Competition -> Program`);
- один спортсмен може мати багато програм (`Skater -> Program`);
- одна програма має багато оцінок від суддів (`Program -> Score`);
- один суддя може виставити багато оцінок (`User -> Score`).

Ця структура забезпечує коректне відображення процесу змагань та дозволяє масштабувати систему для складніших сценаріїв оцінювання.

### 3.4 Реалізація таблиць БД

```
-- CREATE DATABASE
CREATE DATABASE figure_skating;
GO

USE scoring_figure_skating_db;
GO

-- USERS
CREATE TABLE users (
    id INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL,
    email NVARCHAR(100) NOT NULL UNIQUE,
    password NVARCHAR(255) NOT NULL,
    role NVARCHAR(20) NOT NULL
        CHECK (role IN ('admin', 'judge', 'viewer'))
);

-- COMPETITION
CREATE TABLE competition (
    id INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL,
    date DATE NOT NULL,
    location NVARCHAR(255),
    created_by INT,

    FOREIGN KEY (created_by) REFERENCES users(id)
);

-- SKATER
CREATE TABLE skater (
    id INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL,
    country NVARCHAR(100),
    gender NVARCHAR(10)
        CHECK (gender IN ('male', 'female'))
);

-- PROGRAM
CREATE TABLE program (
    id INT IDENTITY(1,1) PRIMARY KEY,
    skater_id INT NOT NULL,
    competition_id INT NOT NULL,
    type NVARCHAR(10) NOT NULL
        CHECK (type IN ('short', 'free')),

    FOREIGN KEY (skater_id) REFERENCES skater(id),
    FOREIGN KEY (competition_id) REFERENCES competition(id)
);

-- SCORE
CREATE TABLE score (
    id INT IDENTITY(1,1) PRIMARY KEY,
    judge_id INT NOT NULL,
    program_id INT NOT NULL,

    technical_score DECIMAL(5,2),
    components_score DECIMAL(5,2),
    deductions DECIMAL(5,2),

    FOREIGN KEY (judge_id) REFERENCES users(id),
    FOREIGN KEY (program_id) REFERENCES program(id),

    CONSTRAINT unique_judge_program UNIQUE (judge_id, program_id)
);
```
![](/assets/labs/lab-2/db.png)

---

## 4. Реалізація backend-частини

### 4.1 Структура серверної частини
![](/assets/labs/lab-2/backend_structure.png)

### 4.2 Підключення до MySQL через ORM SEQUELIZE

```
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  'scoring_figure_skating',
  'sa',
  '',
  {
    host: 'localhost',
    dialect: 'mssql',
    dialectOptions: {
      options: {
        trustServerCertificate: true,
      },
    },
  },
);

module.exports = sequelize;
```

### 4.3 CRUD-маршрути:
Для сутності `Competition` реалізовано повний набір основних операцій:
- `GET /competitions` — отримання списку змагань;
- `GET /competitions/:id` — отримання одного змагання;
- `POST /competitions` — створення нового змагання;
- `PUT /competitions/:id` — оновлення змагання;
- `DELETE /competitions/:id` — видалення змагання (soft delete через зміну статусу).

---

## 5. Прямі SQL-запити з Node.js

### 5.1 Приклад INSERT-запиту (створення змагання)
```
INSERT INTO competition (name, date, location, created_by, category, segment, status)
VALUES ('World Championship 2026', '2026-03-15', 'Boston, USA', 1, 'men_single', 'short_program', 'upcoming');
```

### 5.2 Приклад SELECT-запиту
```
SELECT id, name, date, location, category, segment, status
FROM competition
WHERE status != 'deleted';
```

### 5.3 Приклад UPDATE-запиту
```
UPDATE competition
SET name = 'Updated Championship',
    location = 'Helsinki, Finland',
    status = 'in_progress'
WHERE id = 1;
```

### 5.4 Приклад DELETE-запиту 
```
DELETE FROM score
WHERE program_id IN (
  SELECT id FROM program WHERE competition_id = 1
);

DELETE FROM program
WHERE competition_id = 1;

DELETE FROM competition
WHERE id = 1;
```

---

## 6. Інтеграція з web-

### 6.1 Сторінка зі списком змагань
Маршрут `/competitions` отримує список змагань із backend API і відображає їх у вигляді карток. На цій сторінці реалізовано пошук за назвою змагання, а також можливість створення, редагування та видалення записів. Таким чином перевірка `GET /competitions` виконується безпосередньо через UI без використання Postman.

### 6.2 Детальна сторінка змагання (start order / виступи)
Маршрут `/competition/:id` використовується для перегляду конкретного змагання та пов’язаних із ним даних (список стартового порядку або програм виступів). Через цю сторінку відбувається робота з сутністю `Program`, що дозволяє перевіряти зв’язки між `Competition`, `Skater` і `Program` у реальному інтерфейсі.

### 6.3 Введення та оцінювання виступів
Маршрут `/score/:id` реалізує інтерфейс для суддівського оцінювання виступу. Через цю сторінку суддя вводить:
- technical score;
- components score;
- deductions.

Після відправки даних виконується `POST /results`, що створює або оновлює запис у таблиці score. Це дозволяє перевірити роботу CRUD-операцій для оцінювання без прямого доступу до бази даних.

### 6.4 Результати змагань
Маршрут `/results` відображає загальну таблицю результатів усіх змагань. Дані агрегуються з таблиць program та `score`. Додатково є сторінка `/results/:id`, яка показує детальні результати конкретного змагання, включаючи оцінки кожного судді та підсумковий бал.

### 6.5 Довідники змагань та спортсменів
Окремі сторінки дозволяють працювати з базовими сутностями системи:
- список змагань (`/competitions`);
- список спортсменів (як частина програми стартового порядку);
- формування програм виступів.

Це забезпечує повну інтеграцію frontend і backend та дозволяє демонструвати всі CRUD-операції безпосередньо через інтерфейс користувача.

---

## 7. Повний шлях обробки сутності Competition
У системі оцінювання фігурного катання сутність Competition проходить повний цикл обробки — від створення в інтерфейсі користувача до збереження в базі даних і повернення назад у UI.

### 7.1 Створення змагання у frontend
Користувач відкриває сторінку /competitions і натискає кнопку “New Competition”.
Після цього відкривається модальне вікно CreateCompetitionModal, де вводяться дані:
- назва змагання;
- дата;
- локація;
- категорія;
- сегмент;
- статус.

Після підтвердження форми викликається API-запит: `POST /competitions`

### 7.2 Відправка запиту з frontend
Файл api.js:
```
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3001',
});

export const getCompetitions = () => api.get('/competitions');
export const createCompetition = (data) => api.post('/competitions', data);
export const updateCompetition = (id, data) =>
  api.put(`/competitions/${id}`, data);
export const deleteCompetition = (id) => api.delete(`/competitions/${id}`);

export default api;
```

### 7.3 Обробка запиту на backend (Express)
Запит потрапляє в маршрут: `POST /competitions`
server/routes/competitionRoutes.js:
```
const express = require('express');
const router = express.Router();

const controller = require('../controllers/competitionController');

router.post('/', controller.createCompetition);
router.get('/', controller.getCompetitions);
router.get('/:id', controller.getCompetitionById);
router.put('/:id', controller.updateCompetition);
router.delete('/:id', controller.deleteCompetition);

module.exports = router;
```

server/controllers/competitionController.js:
```
exports.createCompetition = async (req, res) => {
  try {
    const { name, date, location, category, segment, status } = req.body;

    if (!name || !date) {
      return res.status(400).json({
        message: 'Name and date are required',
      });
    }

    const competition = await Competition.create({
      name,
      date,
      location,
      category,
      segment,
      status,
      created_by: 1,
    });

    res.status(201).json(competition);
  } catch (err) {
    console.error(err);
    res.status(500).json({
      message: 'Error creating competition',
    });
  }
};
```

Контролер викликає Sequelize: `Competition.create(req.body)`

### 7.4 Робота ORM (Sequelize)
Sequelize перетворює JavaScript-об’єкт у SQL-запит:
```
INSERT INTO competition (name, date, location, category, segment, status, created_by)
VALUES (...)
```

Після виконання SQL-запиту MSSQL повертає створений запис із id.

### 7.5 Збереження в базі даних

Запис потрапляє в таблицю: `competition` і зберігається разом із всіма зв’язаними полями.

### 7.6 Повернення відповіді у frontend
Backend повертає створений об’єкт:
```
{
  "id": 1,
  "name": "...",
  "date": "...",
  "location": "...",
  "category": "...",
  "segment": "...",
  "status": "upcoming"
}
```

Frontend отримує відповідь і оновлює state: `setCompetitions([res.data, ...competitions]);`

### 7.7 Відображення у UI
Компонент `Competition.jsx` рендерить нове змагання як картку.
```
const handleCreate = async () => {
    try {
      if (form.id) {
        const res = await updateCompetition(form.id, form);

        setCompetitions(
          competitions.map((c) => (c.id === form.id ? res.data : c)),
        );
      } else {
        const res = await createCompetition(form);

        setCompetitions([res.data, ...competitions]);
      }

      setForm({
        name: '',
        date: '',
        location: '',
        category: 'men_single',
        segment: 'short_program',
        status: 'upcoming',
      });

      setOpen(false);
    } catch (err) {
      console.error(err);
    }
  };
```

Дані відображаються у вигляді:
- назва
- дата
- локація
- категорія
- сегмент
- статус

### 7.8 Повний цикл CRUD для Competition
Аналогічно реалізовані інші операції:
- Read → GET /competitions
- Update → PUT /competitions/:id
- Delete → DELETE /competitions/:id

---

## 8. Висновки

У межах лабораторної роботи розроблено систему оцінювання змагань з фігурного катання з використанням Microsoft SQL Server та Node.js. Для роботи з базою даних застосовано ORM Sequelize, через який реалізовано основні CRUD-операції для сутностей змагань, спортсменів, програм і оцінок. Додатково реалізовано виконання прямих SQL-запитів, що дозволяє працювати з базою даних без ORM та демонструє каскадні операції між зв’язаними таблицями. Backend інтегровано з React-інтерфейсом, що забезпечує повний цикл роботи з даними: створення, перегляд, редагування та видалення змагань безпосередньо через UI. Отримана система є базою для подальшого розвитку, зокрема додавання суддівського оцінювання, стартового порядку та розширеної логіки результатів.

---

## 9. Перелік використаних джерел
1. Документація MYSQL.  
2. Документація Node.js.  
3. Документація ORM SEQUELIZE.