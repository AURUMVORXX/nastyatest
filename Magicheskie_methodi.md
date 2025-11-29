Магические методы начинаются с двойного подчеркивания ``__`` и автоматически вызываются в определенных ситуациях.

Магические методы можно также указать public, private или protected, и тогда определенные операции станут недоступны. Например, если конструктор объявить private, то создание новых экземпляров станет недоступно.

### Когда использовать магические методы
- ``__construct``/__destruct`` - инициализация и очистка ресурсов
- ``__get``/``__set`` - для динамических свойств, ORM, Active Record
- ``__call``/``__callStatic`` - для прокси, API клиентов, динамических методов
- ``__toString`` - для удобного вывода объектов
- ``__invoke`` - когда объект должен вести себя как функция
- ``__debugInfo`` - для безопасности при отладке
- ``__sleep``/``__wakeup`` - для кастомизации сериализации

### Важные предупреждения
- **Производительность** - магические методы медленнее прямых вызовов
- **Читаемость** - могут сделать код менее понятным
- **Отладка** - сложнее отслеживать вызовы
- **Используйте умеренно** - только когда действительно нужно

### Основные магические методы

---
#### ``__construct()`` - Конструктор
Вызывается при создании экземпляра класса
```php
<?php
class User {
    public function __construct($name, $email) {
        $this->name = $name;
        $this->email = $email;
        echo "Создан пользователь: $name\n";
    }
}

$user = new User("John", "john@test.com");
// Вывод: "Создан пользователь: John"
?>
```
---
#### ``__destruct()`` - Деструктор
Вызывается при удалении объекта
```php
<?php
class FileHandler {
    private $file;
    
    public function __construct($filename) {
        $this->file = fopen($filename, 'w');
        echo "Файл открыт\n";
    }
    
    public function __destruct() {
        if ($this->file) {
            fclose($this->file);
            echo "Файл закрыт\n";
        }
    }
}

$handler = new FileHandler("test.txt");
// Когда объект уничтожается (скрипт завершается или unset()):
// Вывод: "Файл закрыт"
?>
```
---
#### ``__get()`` и ``__set()`` - Геттеры и сеттеры
Вызываются когда пытаемся прочитать или изменить поле в классе
```php
<?php
class Person {
    private $data = [];
    
    public function __get($name) {
        if (array_key_exists($name, $this->data)) {
            return $this->data[$name];
        }
        return "Свойство $name не существует";
    }
    
    public function __set($name, $value) {
        $this->data[$name] = $value;
    }
    
    public function __isset($name) {
        return isset($this->data[$name]);
    }
    
    public function __unset($name) {
        unset($this->data[$name]);
    }
}

$person = new Person();
$person->name = "Alice";      // __set('name', 'Alice')
$person->age = 25;           // __set('age', 25)

echo $person->name;          // __get('name') -> "Alice"
echo $person->email;         // __get('email') -> "Свойство email не существует"

var_dump(isset($person->name)); // __isset('name') -> true
unset($person->age);          // __unset('age')
?>
```
---
#### ``__call()`` и ``__callStatic()``
Вызывается, когда пытаемся вызвать методы класса
```php
<?php
class Math {
    // Для вызовов методов объекта
    public function __call($method, $arguments) {
        if ($method === 'sum') {
            return array_sum($arguments);
        }
        throw new Exception("Метод $method не существует");
    }
    
    // Для статических вызовов
    public static function __callStatic($method, $arguments) {
        if ($method === 'multiply') {
            return array_product($arguments);
        }
        throw new Exception("Статический метод $method не существует");
    }
}

$math = new Math();
echo $math->sum(1, 2, 3, 4);        // 10 - __call()
echo Math::multiply(2, 3, 4);       // 24 - __callStatic()
?>
```
#### ``__toString()`` - Преобразование в строку
Вызывается, когда пытаемся преобразовать экземпляр класса в строку
```php
<?php
class Product {
    public $name;
    public $price;
    
    public function __construct($name, $price) {
        $this->name = $name;
        $this->price = $price;
    }
    
    public function __toString() {
        return "Товар: {$this->name}, Цена: {$this->price} руб.";
    }
}

$product = new Product("iPhone", 99900);
echo $product; // "Товар: iPhone, Цена: 99900 руб."
?>
```
---
#### ``__invoke()`` - Вызов объекта как функцию
Вызывается, когда пытаемся вызвать экземпляр объекта, как функцию
```php
<?php
class Multiplier {
    private $factor;
    
    public function __construct($factor) {
        $this->factor = $factor;
    }
    
    public function __invoke($number) {
        return $number * $this->factor;
    }
}

$double = new Multiplier(2);
$triple = new Multiplier(3);

echo $double(5);  // 10 - объект вызывается как функция
echo $triple(5);  // 15
?>
```
---
#### ``__debugInfo()`` - Кастомизация var_dump()
Вызывается, когда пытаемся получить дебаг инфу об экземпляре класса
```php
<?php
class SecretData {
    private $public = "Public info";
    private $secret = "Top secret password";
    private $hidden = "Hidden data";
    
    public function __debugInfo() {
        return [
            'public' => $this->public,
            'secret' => '***HIDDEN***',
            'info' => 'Доступно только публичное свойство'
        ];
    }
}

$data = new SecretData();
var_dump($data);
// Вывод только тех данных, которые мы указали в __debugInfo()
?>
```