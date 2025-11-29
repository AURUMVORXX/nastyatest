## MVC (Model-View-Controller)

Преимущества MVC:
- **Разделение ответственности** - каждая часть решает свою задачу
- **Повторное использование кода**
- **Упрощение тестирования** - можно тестировать компоненты отдельно
- **Гибкость** - легко вносить изменения
- **Масштабируемость** - удобно добавлять новый функционал

**Прим. от меня:** Это типа такая структура файлов и функционала, при котором у тебя компоненты логически разделены и минимально зависят друг от друга
---
### Базовая структура MVC
```php
app/
├── Model/
│   └── UserModel.php
├── View/
│   └── user_view.php
├── Controller/
│   └── UserController.php
└── index.php
```
---
### 1. Model (Модель)
```php
<?php
// Model/UserModel.php
class UserModel {
    private $db;

    public function __construct($database) {
        $this->db = $database;
    }

    public function getAllUsers() {
        return $this->db->query("SELECT * FROM users");
    }

    public function getUserById($id) {
        return $this->db->query("SELECT * FROM users WHERE id = ?", [$id]);
    }

    public function createUser($data) {
        return $this->db->query(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            [$data['name'], $data['email']]
        );
    }
}
?>
```
---
### 2. View (Представление)
```php
<?php
// View/user_view.php
class UserView {
    public function renderUsers($users) {
        echo "<h1>User List</h1>";
        echo "<ul>";
        foreach ($users as $user) {
            echo "<li>{$user['name']} - {$user['email']}</li>";
        }
        echo "</ul>";
    }

    public function renderUser($user) {
        echo "<h1>User Details</h1>";
        echo "<p>Name: {$user['name']}</p>";
        echo "<p>Email: {$user['email']}</p>";
    }
}
?>
```
---
### 3. Controller (Контроллер)
```php
<?php
// Controller/UserController.php
class UserController {
    private $model;
    private $view;

    public function __construct($model, $view) {
        $this->model = $model;
        $this->view = $view;
    }

    public function listUsers() {
        $users = $this->model->getAllUsers();
        $this->view->renderUsers($users);
    }

    public function showUser($id) {
        $user = $this->model->getUserById($id);
        $this->view->renderUser($user);
    }

    public function createUser($data) {
        $this->model->createUser($data);
        // Редирект или вывод сообщения
    }
}
?>
```
---
### 4. Front Controller (index.php)
```php
<?php
// index.php - единая точка входа
require_once 'Model/UserModel.php';
require_once 'View/UserView.php';
require_once 'Controller/UserController.php';

// Простая маршрутизация
$action = $_GET['action'] ?? 'list';
$id = $_GET['id'] ?? null;

// Инициализация компонентов
$database = new PDO("mysql:host=localhost;dbname=test", "user", "pass");
$model = new UserModel($database);
$view = new UserView();
$controller = new UserController($model, $view);

// Вызов действия
switch ($action) {
    case 'list':
        $controller->listUsers();
        break;
    case 'show':
        $controller->showUser($id);
        break;
    case 'create':
        $controller->createUser($_POST);
        break;
    default:
        echo "Page not found";
}
?>
```
---
### Современная реализация MVC с роутингом
```php
<?php
// Router.php
class Router {
    private $routes = [];

    public function addRoute($path, $handler) {
        $this->routes[$path] = $handler;
    }

    public function handleRequest($path) {
        if (isset($this->routes[$path])) {
            $handler = $this->routes[$path];
            list($controllerClass, $method) = explode('@', $handler);
            
            require_once "Controller/$controllerClass.php";
            $controller = new $controllerClass();
            $controller->$method();
        } else {
            http_response_code(404);
            echo "Page not found";
        }
    }
}

// Использование
$router = new Router();
$router->addRoute('/users', 'UserController@list');
$router->addRoute('/users/show', 'UserController@show');

$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
$router->handleRequest($path);
?>
```