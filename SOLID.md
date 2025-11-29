**SOLID** - это 5 принципов объектно-ориентированного программирования, которые делают код более гибким, поддерживаемым и расширяемым. - это 5 принципов объектно-ориентированного программирования, которые делают код более гибким, поддерживаемым и расширяемым.

---
#### SRP - Принцип единственной ответственности
Один класс должен иметь только одну причину для изменения
```php
<?php
// ❌ Плохо - класс делает слишком много
class User {
    public function createUser($data) {
        // создание пользователя
    }
    
    public function sendEmail($message) {
        // отправка email
    }
    
    public function logActivity($action) {
        // логирование
    }
}

// ✅ Хорошо - разделение ответственности
class User {
    public function createUser($data) {
        // только создание пользователя
    }
}

class EmailService {
    public function sendEmail($message) {
        // отправка email
    }
}

class Logger {
    public function logActivity($action) {
        // логирование
    }
}
?>
```
---
#### OCP - Принцип открытости/закрытости
Классы должны быть открыты для расширения, но закрыты для модификации
```php
<?php
// ❌ Плохо - при добавлении нового типа нужно менять класс (типа в IF вписывать каждый новый класс)
class AreaCalculator {
    public function calculate($shape) {
        if ($shape instanceof Rectangle) {
            return $shape->width * $shape->height;
        } elseif ($shape instanceof Circle) {
            return pi() * $shape->radius * $shape->radius;
        }
    }
}

// ✅ Хорошо - используем интерфейс
interface Shape {
    public function area();
}

class Rectangle implements Shape {
    public function area() {
        return $this->width * $this->height;
    }
}

class Circle implements Shape {
    public function area() {
        return pi() * $this->radius * $this->radius;
    }
}

class AreaCalculator {
    public function calculate(Shape $shape) {
        return $shape->area();
    }
}
?>
```
---
#### LSP - Принцип подстановки Барбары Лисков
Подклассы должны быть заменяемы своими базовыми классами
```php
<?php
// ❌ Плохо - подкласс нарушает поведение базового
class Bird {
    public function fly() {
        return "Flying";
    }
}

class Penguin extends Bird {
    public function fly() {
        throw new Exception("Penguins can't fly!"); // Нарушение LSP
    }
}

// ✅ Хорошо - правильная иерархия
class Bird {
    // общие методы для всех птиц
}

class FlyingBird extends Bird {
    public function fly() {
        return "Flying";
    }
}

class Penguin extends Bird {
    // пингвины не летают, но не нарушают контракт
}

class Sparrow extends FlyingBird {
    public function fly() {
        return "Sparrow flying";
    }
}
?>
```
---
#### ISP - Принцип разделения интерфейсов
Много специализированных интерфейсов лучше одного универсального
```php
<?php
// ❌ Плохо - один большой интерфейс
interface Worker {
    public function work();
    public function eat();
    public function sleep();
}

class Robot implements Worker {
    public function work() { /* OK */ }
    public function eat() { /* Бессмысленно для робота */ }
    public function sleep() { /* Бессмысленно для робота */ }
}

// ✅ Хорошо - разделенные интерфейсы
interface Workable {
    public function work();
}

interface Eatable {
    public function eat();
}

interface Sleepable {
    public function sleep();
}

class Human implements Workable, Eatable, Sleepable {
    public function work() { /* ... */ }
    public function eat() { /* ... */ }
    public function sleep() { /* ... */ }
}

class Robot implements Workable {
    public function work() { /* ... */ }
}
?>
```
---
#### DIP - Принцип инверсии зависимостей
Зависимость от абстракций, а не от конкретных реализаций (типа, проверяем что класс имеет определенный функционал,метод, а не название)
```php
<?php
// ❌ Плохо - зависимость от конкретного класса
class MySQLDatabase {
    public function query($sql) {
        // MySQL запрос
    }
}

class UserService {
    private $database;
    
    public function __construct() {
        $this->database = new MySQLDatabase(); // Жесткая зависимость
    }
}

// ✅ Хорошо - зависимость от интерфейса
interface Database {
    public function query($sql);
}

class MySQLDatabase implements Database {
    public function query($sql) {
        // MySQL запрос
    }
}

class PostgreSQLDatabase implements Database {
    public function query($sql) {
        // PostgreSQL запрос
    }
}

class UserService {
    private $database;
    
    public function __construct(Database $database) { // Инъекция зависимости
        $this->database = $database;
    }
}

// Использование
$database = new MySQLDatabase();
$userService = new UserService($database);
?>
```
