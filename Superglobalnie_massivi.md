Суперглобальные массивы доступны в любой области видимости и содержат информацию о запросе, сервере, сессиях и т.д.

### Основные суперглобальные массивы

---
#### ``$_GET`` - параметры GET запроса
```php
<?php
// URL: http://site.com/?name=John&age=25&page=1

// Получение значений
$name = $_GET['name'] ?? 'Guest'; // John
$age = $_GET['age'] ?? 0;         // 25
$page = $_GET['page'] ?? 1;       // 1

// Безопасное получение с проверкой
if (isset($_GET['search'])) {
    $search = htmlspecialchars($_GET['search']);
    echo "Поиск: $search";
}

// Перебор всех GET параметров
foreach ($_GET as $key => $value) {
    echo "Параметр: $key = $value<br>";
}

// Пример формы
?>
<form method="GET" action="">
    <input type="text" name="username" placeholder="Имя">
    <input type="email" name="email" placeholder="Email">
    <button type="submit">Отправить</button>
</form>
```
---
#### ``$_POST`` - данные POST запроса
```php
<?php
// Обработка формы
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Безопасное получение данных
    $username = htmlspecialchars($_POST['username'] ?? '');
    $email = filter_var($_POST['email'] ?? '', FILTER_VALIDATE_EMAIL);
    $password = $_POST['password'] ?? '';
    
    if ($email) {
        echo "Добро пожаловать, $username! Email: $email";
    } else {
        echo "Неверный email!";
    }
}
?>

<form method="POST" action="">
    <input type="text" name="username" required placeholder="Имя пользователя">
    <input type="email" name="email" required placeholder="Email">
    <input type="password" name="password" required placeholder="Пароль">
    <select name="country">
        <option value="ru">Россия</option>
        <option value="us">США</option>
    </select>
    <button type="submit">Регистрация</button>
</form>
```
---
#### ```$_REQUEST`` - объединение GET, POST и COOKIE
```php
<?php
// Содержит данные из $_GET, $_POST и $_COOKIE
// (порядок определяется php.ini - request_order)

$action = $_REQUEST['action'] ?? 'default';

// Осторожно! Может содержать данные из разных источников
$user_input = $_REQUEST['data'] ?? '';

// Пример использования для AJAX запросов
if (isset($_REQUEST['ajax'])) {
    header('Content-Type: application/json');
    echo json_encode(['status' => 'success', 'data' => $_REQUEST]);
    exit;
}
?>
```
---
#### ``$_SERVER`` - информация о сервере и запросе
```php
<?php
// Основные элементы $_SERVER
echo "HTTP метод: " . $_SERVER['REQUEST_METHOD'] . "<br>";
echo "URL запроса: " . $_SERVER['REQUEST_URI'] . "<br>";
echo "IP клиента: " . $_SERVER['REMOTE_ADDR'] . "<br>";
echo "User Agent: " . $_SERVER['HTTP_USER_AGENT'] . "<br>";
echo "Реферер: " . ($_SERVER['HTTP_REFERER'] ?? 'нет') . "<br>";
echo "Сервер: " . $_SERVER['SERVER_NAME'] . "<br>";
echo "Порт: " . $_SERVER['SERVER_PORT'] . "<br>";

// Определение базового URL
$protocol = isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] === 'on' ? 'https' : 'http';
$base_url = $protocol . '://' . $_SERVER['HTTP_HOST'] . '/';

// Простой роутинг
$request_uri = $_SERVER['REQUEST_URI'];
if (strpos($request_uri, '/admin') === 0) {
    // Админка
    include 'admin.php';
} elseif (strpos($request_uri, '/api') === 0) {
    // API
    include 'api.php';
} else {
    // Сайт
    include 'website.php';
}
?>
```
---
#### ``$_SESSION`` - работа с сессиями
В любом участке кода можно создать сессию, и писать данные туда. И потом данные прочитать в другом файле, например.
```php
<?php
// Начало сессии
session_start();

// Сохранение данных в сессию
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'john_doe';
$_SESSION['last_visit'] = date('Y-m-d H:i:s');

// Получение данных из сессии
if (isset($_SESSION['user_id'])) {
    echo "Привет, " . $_SESSION['username'];
    echo "Ваш последний визит: " . $_SESSION['last_visit'];
}

// Удаление данных из сессии
unset($_SESSION['last_visit']);

// Полное уничтожение сессии
session_destroy();

// Пример аутентификации
function login($user_id, $username) {
    session_start();
    $_SESSION['user_id'] = $user_id;
    $_SESSION['username'] = $username;
    $_SESSION['logged_in'] = true;
}

function checkAuth() {
    session_start();
    return isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
}

function logout() {
    session_start();
    session_destroy();
    header('Location: login.php');
    exit;
}
?>
```
---
#### ``$_COOKIE`` - работа с cookies
Cookie - это хранилище данных прямо в браузере клиента. Эти данные отправляются тебе в код автоматически, когда пользователь заходит на твой сайт.
```php
<?php
// Установка cookie
setcookie('user_preference', 'dark_theme', time() + 3600 * 24 * 30, '/');
setcookie('visit_count', '1', time() + 3600 * 24 * 365, '/');

// Чтение cookie
$theme = $_COOKIE['user_preference'] ?? 'light_theme';
$visit_count = $_COOKIE['visit_count'] ?? 0;

// Увеличение счетчика посещений
if (!isset($_COOKIE['visit_count'])) {
    setcookie('visit_count', '1', time() + 3600 * 24 * 365, '/');
} else {
    $new_count = intval($_COOKIE['visit_count']) + 1;
    setcookie('visit_count', $new_count, time() + 3600 * 24 * 365, '/');
}

// Удаление cookie
setcookie('user_preference', '', time() - 3600, '/');

// Пример: запомнить пользователя
if (isset($_COOKIE['remember_me'])) {
    $user_id = $_COOKIE['remember_me'];
    // Автоматический вход пользователя
}
?>
```
---
#### ``$_FILES`` - работа с загружаемыми файлами
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['file_upload'])) {
    $file = $_FILES['file_upload'];
    
    // Информация о файле
    $filename = $file['name'];
    $tmp_name = $file['tmp_name'];
    $size = $file['size'];
    $error = $file['error'];
    
    // Проверки
    if ($error === UPLOAD_ERR_OK) {
        // Проверка типа файла
        $allowed_types = ['image/jpeg', 'image/png', 'image/gif'];
        $file_type = mime_content_type($tmp_name);
        
        if (in_array($file_type, $allowed_types) && $size < 5 * 1024 * 1024) {
            // Сохранение файла
            $upload_dir = 'uploads/';
            $new_filename = uniqid() . '_' . $filename;
            move_uploaded_file($tmp_name, $upload_dir . $new_filename);
            echo "Файл успешно загружен!";
        } else {
            echo "Недопустимый тип файла или размер слишком большой!";
        }
    } else {
        echo "Ошибка загрузки: $error";
    }
}
?>

<form method="POST" enctype="multipart/form-data">
    <input type="file" name="file_upload" accept="image/*">
    <button type="submit">Загрузить файл</button>
</form>
```
---
#### ``$_ENV`` - переменные окружения
```php
<?php
// Чтение из .env файла (обычно через библиотеку)
$db_host = $_ENV['DB_HOST'] ?? 'localhost';
$db_name = $_ENV['DB_NAME'] ?? 'myapp';
$db_user = $_ENV['DB_USER'] ?? 'root';
$db_pass = $_ENV['DB_PASS'] ?? '';

// Подключение к базе данных
try {
    $pdo = new PDO(
        "mysql:host=$db_host;dbname=$db_name",
        $db_user,
        $db_pass
    );
} catch (PDOException $e) {
    die("Ошибка подключения: " . $e->getMessage());
}

// Все переменные окружения
foreach ($_ENV as $key => $value) {
    echo "$key: $value<br>";
}
?>
```
### Пример - простая система авторизации
```php
<?php
// auth.php
session_start();

class Auth {
    public static function login($user_id, $username) {
        $_SESSION['user_id'] = $user_id;
        $_SESSION['username'] = $username;
        $_SESSION['logged_in'] = true;
        $_SESSION['login_time'] = time();
    }
    
    public static function logout() {
        session_destroy();
        header('Location: login.php');
        exit;
    }
    
    public static function isLoggedIn() {
        return isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
    }
    
    public static function requireAuth() {
        if (!self::isLoggedIn()) {
            header('Location: login.php');
            exit;
        }
    }
}

// protected_page.php
Auth::requireAuth();
echo "Добро пожаловать, " . $_SESSION['username'];
?>
```