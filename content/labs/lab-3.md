## 1. Тема, мета, посилання

### 1.1 Тема
«Реєстрація та авторизація користувачів у веб-застосунку. Захист маршрутів. Робота з токеном доступу та профілем користувача».

### 1.2 Мета
Реалізувати в проєкті Scoring figure skating app модуль автентифікації та базової авторизації: створення облікового запису, вхід у систему, отримання даних поточного користувача, оновлення профілю, зміну пароля, вихід із системи, а також захист окремих маршрутів на frontend і backend.

### 1.3 Посилання
- Репозиторій власного веб-застосунку (GitHub): [посилання](https://github.com/hannoveran/scoring-figure-skating-app.git)
- Власний веб-застосунок (Жива сторінка): [посилання](https://hannoveran.github.io/scoring-figure-skating-app/)
- Репозиторій звітного HTML-документа (GitHub): [посилання](https://github.com/hannoveran/IC-31_appRECORD-MamchukAnna-FIOT-2026.git)
- Звітний HTML-документ (Жива сторінка): [посилання](https://hannoveran.github.io/IC-31_appRECORD-MamchukAnna-FIOT-2026/)

---

## 2. Короткі теоретичні відомості

### 2.1 Автентифікація та авторизація
У web-застосунках необхідно відрізняти дві базові задачі. **Автентифікація** відповідає на питання, хто саме працює із системою, а **авторизація** визначає, які саме дії цей користувач має право виконувати. Для helpdesk-системи це критично, оскільки звичайний користувач, агент підтримки та адміністратор мають різні ролі й різні сценарії роботи.

### 2.2 Токенний підхід
У межах цієї лабораторної роботи використано підхід з **access token**, який видається після реєстрації або входу. Далі клієнт передає цей токен у заголовку `Authorization: Bearer <token>` під час звернення до захищених маршрутів API. Це дозволяє відокремити frontend-частину від backend-логіки перевірки доступу.

### 2.3 Захист паролів
Пароль не повинен зберігатися у відкритому вигляді. У реалізації проєкту для цього використано хешування пароля перед збереженням у базі даних. Такий підхід зменшує ризики витоку чутливих даних і є базовою вимогою для будь-якої системи з обліковими записами.

### 2.4 Захищені маршрути
Окрім перевірки токена на рівні API, у frontend-частині важливо не дозволяти неавторизованому користувачу потрапляти на сторінки профілю. Для цього застосовується middleware-перевірка, яка перенаправляє користувача на сторінку входу або, навпаки, не дозволяє вже авторизованому користувачу повторно заходити на сторінки `/login` та `/register`.

---

## 3. Реалізований функціонал Lab 3

### 3.1 Основні сценарії
У межах лабораторної роботи в проєкті Scoring Figure Skating App реалізовано такі основні сценарії:
- реєстрація нового користувача;
- вхід у систему за email і паролем;
- отримання інформації про поточного користувача (/me);
- редагування профілю (ім’я та email);
- зміна пароля;
- вихід із системи;
- перевірка токена та захист API-маршрутів на backend.

### 3.2 Ролі користувачів
У моделі даних використовується система ролей, яка відповідає предметній області застосунку:
- `admin` — створює змагання, керує системою;
- `judge` — виставляє оцінки спортсменам;
- `viewer` — переглядає результати.

Роль користувача зберігається в базі даних і передається в токені доступу. Це дозволяє реалізувати розмежування прав доступу до різних функцій системи як на backend, так і на frontend.

### 3.3 Захищені маршрути API
Для модуля автентифікації реалізовано такі маршрути:
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/refresh`
- `GET /api/auth/me`
- `PATCH /api/auth/profile`
- `POST /api/auth/change-password`
- `POST /api/auth/logout`

Маршрути me, profile та change-password є захищеними і доступні лише за наявності валідного JWT токена (Bearer token).

Окрім цього, реалізовано механізм оновлення короткоживучого accessToken за допомогою refreshToken, який зберігається в cookie. Це дозволяє підтримувати сесію користувача без повторного введення логіну і пароля.

---

## 4. Реалізація backend-частини

### 4.1 Структура auth-модуля
Основна логіка автентифікації у проєкті Scoring Figure Skating App реалізована в таких файлах:
- `apps/server/routes/authRoutes.js` — HTTP-маршрути для реєстрації, входу, профілю, зміни пароля та виходу;
- `apps/server/controllers/authController.js` — бізнес-логіка (JWT, хешування, робота з користувачем);
- `apps/server/middleware/authMiddleware.js` — middleware для перевірки токена;
- `apps/server/models/User.js` — модель користувача (Sequelize);
- `apps/server/config/db.js` — підключення до бази даних MSSQL;
- `apps/server/app.js` — підключення маршрутів і middleware.

### 4.2 Розширення моделі користувача
Для підтримки автентифікації використовується модель користувача Sequelize:

```
const { DataTypes } = require('sequelize');
const sequelize = require('../config/db');

const User = sequelize.define(
  'User',
  {
    name: DataTypes.STRING,
    email: DataTypes.STRING,
    password: DataTypes.STRING,
    role: DataTypes.STRING,
    refreshToken: DataTypes.STRING,
  },
  {
    tableName: 'users',
    timestamps: false,
  },
);

module.exports = User;
```

Поле `email` є унікальним на рівні бази даних, що запобігає створенню дублікованих користувачів.

### 4.3 Хешування пароля
Паролі користувачів не зберігаються у відкритому вигляді. Перед записом у базу даних використовується бібліотека `bcrypt`:

```
const hashedPassword = await bcrypt.hash(password, 10);
```
Під час входу пароль перевіряється за допомогою:

```
const isMatch = await bcrypt.compare(password, user.password);
```
Це забезпечує базовий рівень безпеки даних користувачів.

### 4.4 Формування access token
Після успішної реєстрації або входу сервер генерує:
- `accessToken` — короткоживучий JWT (15 хв);
- `refreshToken` — довгоживучий токен (7 днів).

```
const accessToken = jwt.sign(
  { id: user.id, email: user.email, role: user.role },
  JWT_SECRET,
  { expiresIn: '15m' }
);

const refreshToken = jwt.sign(
  { id: user.id },
  JWT_SECRET,
  { expiresIn: '7d' }
);
```

refreshToken зберігається:
- у базі даних (поле `refreshToken`);
- у cookie (`httpOnly`).

### 4.5 Реєстрація користувача
Маршрут `POST /api/auth/register`:
- перевіряє валідність даних через `express-validator`;
- перевіряє унікальність email;
- хешує пароль;
- створює користувача;
- генерує токени.

```
exports.register = async (req, res) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }

  const { name, email, password, passwordConfirmation } = req.body;

  if (password !== passwordConfirmation) {
    return res.status(400).json({ message: 'Passwords do not match' });
  }

  const existingUser = await User.findOne({ where: { email } });

  if (existingUser) {
    return res.status(409).json({ message: 'Email already exists' });
  }

  const hashedPassword = await bcrypt.hash(password, 10);

  const user = await User.create({
    name,
    email,
    password: hashedPassword,
    role: 'viewer',
  });
};
```

### 4.6 Вхід у систему
Маршрут `POST /api/auth/login`:
- знаходить користувача за email;
- перевіряє пароль;
- генерує токени.

```
exports.login = async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ where: { email } });

  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  const isMatch = await bcrypt.compare(password, user.password);

  if (!isMatch) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
};
```

### 4.7 Перевірка токена на захищених маршрутах
Для захищених маршрутів використовується middleware:

```
const jwt = require('jsonwebtoken');

module.exports = function (req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader) {
    return res.status(401).json({ message: 'No token' });
  }

  const token = authHeader.split(' ')[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ message: 'Invalid token' });
  }
};
```

Цей middleware додає `req.user`, який використовується в інших контролерах.

### 4.8 Оновлення access token
Маршрут `POST /api/auth/refresh`:
- читає `refreshToken` з cookie;
- перевіряє його валідність;
- створює новий `accessToken`.

```
exports.refresh = async (req, res) => {
  const token = req.cookies.refreshToken;

  if (!token) {
    return res.status(401).json({ message: 'No refresh token' });
  }

  const decoded = jwt.verify(token, JWT_SECRET);

  const user = await User.findByPk(decoded.id);

  if (!user || user.refreshToken !== token) {
    return res.status(403).json({ message: 'Invalid refresh token' });
  }

  const newAccessToken = jwt.sign(
    { id: user.id, email: user.email, role: user.role },
    JWT_SECRET,
    { expiresIn: '15m' }
  );

  res.json({ accessToken: newAccessToken });
};
```

### 4.9 Робота з профілем і паролем
Після автентифікації користувач може:
- отримати дані через `GET /api/auth/me`;
- оновити профіль через `PATCH /api/auth/profile`;
- змінити пароль через `POST /api/auth/change-password`;
- вийти із системи через `POST /api/auth/logout`.

```
exports.logout = async (req, res) => {
  const token = req.cookies.refreshToken;

  if (token) {
    const user = await User.findOne({ where: { refreshToken: token } });

    if (user) {
      user.refreshToken = null;
      await user.save();
    }
  }

  res.clearCookie('refreshToken');
  res.json({ message: 'Logged out' });
};
```

Під час logout refresh-токен видаляється як з cookie, так і з бази даних, що завершує сесію користувача.

---

## 5. Реалізація frontend-частини

### 5.1 Сторінки автентифікації
У клієнтській частині (React) додано окремі маршрути:
- `/login`
- `/register`

Маршрути описані у `App.jsx`:

```
<Route path="login" element={<Login />} />
<Route path="register" element={<Register />} />
```

Сторінки `Login` і `Register` реалізовані як окремі компоненти з формами та обробкою submit через API.

Приклад логіки для реєстрації:

```
const handleSubmit = async (e) => {
  e.preventDefault();

  await api.post('/api/auth/register', {
    name,
    email,
    password,
    passwordConfirmation,
  });
};
```

Для входу:

```
const handleSubmit = async (e) => {
  e.preventDefault();

  const res = await api.post('/api/auth/login', {
    email,
    password,
  });

  localStorage.setItem('token', res.data.accessToken);
};
```

Після успішної авторизації токен зберігається і використовується для подальших запитів.

### 5.2 Контекст авторизації
Для глобального доступу до стану користувача реалізовано `AuthContext`.

Підключення виконується у main.jsx:

```
import { AuthProvider } from './context/AuthContext';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <AuthProvider>
      <App />
    </AuthProvider>
  </StrictMode>,
);
```

Контекст зберігає:
- поточного користувача
- токен
- функції login / logout

### 5.3 Інтеграція з API
Для роботи з backend використовується Axios.

```
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3001',
});

export default api;
```

Для авторизованих запитів додається токен:

```
api.defaults.headers.common['Authorization'] = `Bearer ${token}`;
```

### 5.4 UI авторизації (Header + BurgerMenu)
Керування авторизацією інтегровано в хедер.

- якщо користувач не авторизований → кнопки Login / Register
- якщо авторизований → показ ролі + кнопка 

```
{user ? (
  <>
    <div className="role">{user.role}</div>
    <button onClick={logout}>Вийти</button>
  </>
) : (
  <>
    <Link to="/login">Login</Link>
    <Link to="/register">Register</Link>
  </>
)}
```

### 5.5 Захист сторінок
На frontend захист реалізовано через перевірку токена:
- якщо токена немає → редирект на /login
- якщо є → доступ дозволено

```
if (!token) {
  navigate('/login');
}
```

---

## 6. Перевірка роботи функціоналу

### 6.1 Типові сценарії перевірки
Перевірено такі сценарії:
- реєстрація нового користувача (admin/judge)
- помилка при дублюванні email (409)
- вхід у систему
- помилка входу (невірний пароль → 401)
- доступ до захищених маршрутів без токена
- доступ із валідним токеном
- logout (очищення токена)
- повторний доступ після logout

### 6.2 Очікувана поведінка
Успішні запити:
- `201` — реєстрація
- `200` — login, me, profile
- повертається `accessToken` і дані користувача

Помилки:
- `400` — валідація (наприклад, password mismatch)
- `401` — неправильні дані або відсутній токен
- `409` — email вже існує

---

## 7. Встановлення необхідних бібліотек

### 7.1 Призначення основних бібліотек
- `express` — сервер і маршрути
- `cors` — доступ із frontend
- `sequelize` — ORM для роботи з БД
- `tedious` — драйвер для SQL Server
- `bcrypt` — хешування паролів
- `jsonwebtoken` — JWT токени
- `dotenv` — змінні середовища (.env)
- `express-validator` — валідація запитів
- `cookie-parser` — робота з cookie (refreshToken)
- `express-rate-limit` — обмеження спроб входу
- `nodemon` — автоперезапуск сервера

---

## 8. Скріншоти результатів

![Сторінка реєстрації](/assets/labs/lab-3/register_page.png)  
**Рис. 1 – Сторінка реєстрації `/register`.**

![Сторінка входу](/assets/labs/lab-3/login_page.png)  
**Рис. 2 – Сторінка входу `/login`.**

![Приклад реєстрації](/assets/labs/lab-3/register_example.png)  
**Рис. 3 – Приклад реєстрації.**

![Приклад входу](/assets/labs/lab-3/login_example.png)  
**Рис. 4 – Приклад входу.**

![Результат авторизації](/assets/labs/lab-3/login_result.png)  
**Рис. 5 – Результат авторизації.**

![Реєстрація користувача у Postman](/assets/labs/lab-3/postman_register.png)  
**Рис. 6 – Успішна реєстрація через `POST /api/auth/register`.**

![Реєстрація з помилкою у Postman](/assets/labs/lab-3/postman_registerValidationError.png)  
**Рис. 7 – Помилка валідації під час `POST /api/auth/register`, коли підтвердження пароля не збігається.**

![Вхід у систему у Postman](/assets/labs/lab-3/postman_login.png)  
**Рис. 8 – Успішний вхід через `POST /api/auth/login`.**

![Помилка входу у Postman](/assets/labs/lab-3/postman_loginError.png)  
**Рис. 9 – Помилка входу з неправильними обліковими даними.**

![Отримання поточного користувача у Postman](/assets/labs/lab-3/postman_getCurrentUser.png)  
**Рис. 10 – Отримання поточного користувача `GET /api/auth/me`**

![Оновлення профілю у Postman](/assets/labs/lab-3/postman_newProfileName.png)  
**Рис. 11 – Оновлення профілю через `PATCH /api/auth/profile`**

![Заміна паролю у Postman](/assets/labs/lab-3/postman_passwordUpdate.png)  
**Рис. 12 – Зміна паролю через `GET /api/auth/me`**

---

## 9. Висновки
У межах лабораторної роботи реалізовано базовий модуль автентифікації та авторизації для веб-застосунку суддівської платформи з фігурного катання. На backend-частині створено маршрути для реєстрації, входу в систему, отримання даних поточного користувача, оновлення профілю, зміни пароля та виходу з системи.

Паролі користувачів зберігаються у базі даних у захешованому вигляді з використанням bcrypt, що забезпечує захист конфіденційних даних. Для авторизації доступу до захищених маршрутів використовується JWT (JSON Web Token), який передається між клієнтом і сервером та перевіряється middleware.

На frontend-частині реалізовано сторінки реєстрації та входу, а також базову інтеграцію з API через axios. Після успішної авторизації токен зберігається на клієнті та використовується для доступу до захищених запитів. Також реалізовано базову структуру інтерфейсу з використанням компонентного підходу React.

---

## 10. Перелік використаних джерел
1. Документація Node.js
2. Документація Express.js
3. Документація Sequelize ORM
4. Документація JSON Web Token (JWT)
5. Документація bcrypt
6. Документація React
7. Документація Axios
8. Документація Microsoft SQL Server