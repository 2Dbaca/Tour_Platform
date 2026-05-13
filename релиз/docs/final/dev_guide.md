# Руководство разработчика
## Веб-платформа для бронирования и управления турами

---

## 1. Структура проекта

```text
tour-platform/
├── .htconfig # Конфигурация
├── docker-compose.yml # Docker оркестрация
├── Dockerfile # Сборка PHP контейнера
├── database.sql # SQL дамп БД
├── index.php # Точка входа (роутер)
├── init_db.php # Инициализация БД данными
├── test.php # Тест PHP
├── test_login.php # Тест входа
├── README.md # Документация
│
├── assets/ # Статические файлы (CSS, JS)
├── config/ # Конфигурация БД
├── controllers/ # Контроллеры (MVC)
├── core/ # Ядро (Router, Session, Database)
├── models/ # Модели данных
├── views/ # Шаблоны страниц
├── docker/ # Docker конфигурации
└── uploads/ # Загруженные файлы
```


**Назначение ключевых папок:**

| Папка | Назначение |
|-------|------------|
| `controllers/` | Обработка запросов, бизнес-логика |
| `models/` | Работа с базой данных |
| `views/` | HTML-шаблоны страниц |
| `core/` | Ядро: маршрутизация, сессии, подключение к БД |
| `assets/` | CSS, JavaScript, изображения |
| `docker/` | Конфигурационные файлы Docker |

---

## 2. Архитектура и паттерны

Приложение построено на **паттерне MVC** (Model-View-Controller):

- **Model** — работа с базой данных (папка `models/`)
- **View** — HTML-шаблоны (папка `views/`)
- **Controller** — бизнес-логика (папка `controllers/`)

### 2.1. Пример кода — контроллер

```php
// controllers/AuthController.php
class AuthController {
    public function login() {
        $login = $_POST["login"] ?? "";
        $password = $_POST["password"] ?? "";
        
        $pdo = new PDO("mysql:host=db;dbname=tour_platform", "root", "rootpassword");
        $stmt = $pdo->prepare("SELECT * FROM users WHERE login = :login OR email = :login");
        $stmt->execute([':login' => $login]);
        $user = $stmt->fetch();
        
        if($user && password_verify($password, $user['password'])) {
            $_SESSION["user_id"] = $user['id'];
            $_SESSION["user_role"] = $user['role'];
            header("Location: /admin");
        } else {
            $_SESSION["login_error"] = "Неверный логин или пароль";
            header("Location: /login");
        }
    }
}
```

### 2.2. Пример кода - модель

```php
// models/User.php
class User {
    private $db;
    
    public function login($login, $password) {
        $sql = "SELECT * FROM users WHERE login = :login OR email = :login";
        $stmt = $this->db->prepare($sql);
        $stmt->execute([':login' => $login]);
        $user = $stmt->fetch();
        
        if ($user && password_verify($password, $user['password'])) {
            return $user;
        }
        return false;
    }
}
```

### 2.3. Маршрутизация (единая точка входа)

Файл index.php обрабатывает все запросы и вызывает нужный контроллер:

```php
// index.php
switch ($url) {
    case "/tours":
        $controller = new TourController();
        $controller->index();
        break;
    case "/admin/reports":
        $controller = new AdminController();
        $controller->reports();
        break;
    case "/login":
        $controller = new AuthController();
        $controller->showLoginForm();
        break;
    // ... другие маршруты
}
```

## 3. Технологический стек и причины выбора

| Технология | Почему выбрали |
|------------|----------------|
| **PHP (нативный, без фреймворков)** | Демонстрация глубокого понимания языка и принципов MVC |
| **PDO (подготовленные запросы)** | Защита от SQL-инъекций |
| **password_hash() / password_verify()** | Безопасное хеширование паролей (bcrypt) |
| **Сессии + проверка ролей** | Разграничение доступа (гость, клиент, админ) |
| **Docker** | Идентичность среды, простота развёртывания |
| **htmlspecialchars()** | Защита от XSS-атак |
| **MySQL** | Надёжность, производительность, бесплатность |

## 4. Возможные ошибки и их устранение

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `PDOException: could not find driver` | Нет расширения PDO MySQL | Установить `php-mysql` |
| `404 Not Found` | Неправильный URL | Проверить маршруты в `index.php` |
| Неверный логин/пароль | Неправильный хеш в БД | Использовать `password_verify()` |
| Ошибка подключения к БД | Контейнер MySQL не запущен | Выполнить `docker-compose up -d` |
| Сессия не сохраняется | Не вызван `session_start()` | Проверить `Session::start()` |

## 5. Работа с Docker

### 5.1. Запуск всех сервисов

```bash
docker-compose up -d
```

### 5.2. Остановка

```bash
docker-compose down
```

### 5.3. Просмотр логов 

```bash
docker-compose logs -f
```

### 5.4. Добавление нового сервиса

Чтобы добавить новый сервис в Docker-окружение:

1. Откройте файл `docker-compose.yml`
2. Добавьте новый сервис в секцию `services`, например:

```yaml
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

3. Сохраните файл и выполните команду:

```bash
docker-compose up -d
```

4. Проверьте, что новый сервис запущен:

```bash
docker-compose ps
```

### 6. Безопасность

### 6.1. Защита от SQL-инъекций

Используются подготовленные запросы PDO с плейсхолдерами:

```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE login = :login");
$stmt->execute([':login' => $login]);
```

### 6.2. Хеширование паролей

Используется password_hash() с алгоритмом PASSWORD_DEFAULT (bcrypt):

```php
$hashedPassword = password_hash($password, PASSWORD_DEFAULT);
```

Для проверки пароля используется password_verify():

```php
if (password_verify($password, $user['password'])) {
    // пароль верный
}
```

### 6.3. Защита от XSS

Все данные при выводе на страницу проходят через htmlspecialchars():

```php
echo htmlspecialchars($user['login']);
```

### 6.4. Проверка авторизации и ролей

Проверка авторизации (доступ только для залогиненных):

```php
if(!isset($_SESSION["user_id"])) { 
    header("Location: /login"); 
    exit; 
}
```

Проверка роли администратора (строгое сравнение ===):

```php
if($_SESSION["user_role"] !== "admin") { 
    header("Location: /"); 
    exit; 
}
```

| Тип проверки | Код | Назначение |
|--------------|-----|------------|
| Авторизация | `if(!isset($_SESSION["user_id"]))` | Проверка, что пользователь вошёл в систему |
| Роль администратора | `if($_SESSION["user_role"] !== "admin")` | Проверка, что у пользователя права админа |

✅ Использование === (три знака равенства) обеспечивает строгое сравнение по значению и типу, что исключает возможность обхода проверки.

