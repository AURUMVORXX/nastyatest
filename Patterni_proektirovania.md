#### Singleton (Одиночка)
Гарантирует, что класс имеет только один экземпляр

**Прим. от меня:** Представь подключение к базе данных в коде. Оно должно быть только одно, но быть доступным везде, в разных файлах. Ты делаешь так, чтобы при первом обращении к классу - создалось подключение, а последующие вызовы просто обращались к уже созданному

```php
<?php
class DatabaseConnection {
    private static $instance = null;
    private $connection;
    
    // Делаем конструктор приватным для запрета создания новых экземпляров извне
    private function __construct() {
        $this->connection = new PDO("mysql:host=localhost;dbname=test", "user", "pass");
    }

    public static function getInstance() {
        if (self::$instance == null) {
            self::$instance = new DatabaseConnection();
        }
        return self::$instance;
    }

    public function getConnection() {
        return $this->connection;
    }

    // Запрещаем клонирование
    private function __clone() {}
}

// Использование
$db1 = DatabaseConnection::getInstance();
$db2 = DatabaseConnection::getInstance();
var_dump($db1 === $db2); // true - это один и тот же объект
?>
```
---
#### Factory (Фабрика)
Создает объекты без указания конкретного класса

**Прим. от меня:** Представь у тебя на фронтенде фанпея есть выбор как оплачивать - СБП, PayPal, карта и т.п. То есть прям строками в меню. И фронтенд тебе это и присылает. Фабрика позволяет просто объявить условие (if - else), которое будет сопостовлять строку с нужным классом. И ты тогда можешь просто вызвать фабрику с той строкой, что тебе прислал фронтенд, и нужный класс для оплаты будет создан.
```php
<?php
interface PaymentMethod {
    public function pay($amount);
}

class CreditCardPayment implements PaymentMethod {
    public function pay($amount) {
        return "Paid $amount via Credit Card";
    }
}

class PayPalPayment implements PaymentMethod {
    public function pay($amount) {
        return "Paid $amount via PayPal";
    }
}

class PaymentFactory {
    public static function create($type) {
        return match($type) {
            'creditcard' => new CreditCardPayment(),
            'paypal' => new PayPalPayment(),
            default => throw new Exception("Unknown payment type")
        };
    }
}

// Использование
$payment = PaymentFactory::create('creditcard');
echo $payment->pay(100);
?>
```
---
#### Observer (Наблюдатель)
Определяет зависимость "один-ко-многим" между объектами

**Прим. от меня:** Представь на сайте лискинс есть несколько способ уведомлений о покупке - на почту, на сайте всплывашкой и какая-нибудь история покупок. Ты можешь создать три разных класса для этих целей, но функцию для уведов внутри назвать одинаково (update например). Тогда, ты можешь все три класса закинуть в функцию, которая генерирует уведомления - она просто вызовет функцию update во всех классах, а какой это класс - ей уже похуй 
```php
<?php
interface Observer {
    public function update($data);
}

interface Subject {
    public function attach(Observer $observer);
    public function detach(Observer $observer);
    public function notify();
}

class NewsPublisher implements Subject {
    private $observers = [];
    private $news;

    public function attach(Observer $observer) {
        $this->observers[] = $observer;
    }

    public function detach(Observer $observer) {
        $this->observers = array_filter($this->observers, function($obs) use ($observer) {
            return $obs !== $observer;
        });
    }

    public function notify() {
        foreach ($this->observers as $observer) {
            $observer->update($this->news);
        }
    }

    public function setNews($news) {
        $this->news = $news;
        $this->notify();
    }
}

class EmailNotifier implements Observer {
    public function update($data) {
        echo "Sending email: $data\n";
    }
}

class SMSNotifier implements Observer {
    public function update($data) {
        echo "Sending SMS: $data\n";
    }
}

// Использование
$publisher = new NewsPublisher();
$publisher->attach(new EmailNotifier());
$publisher->attach(new SMSNotifier());

$publisher->setNews("Breaking news: PHP 8.4 released!");
?>
```
#### Strategy (Стратегия)
Определяет семейство алгоритмов, инкапсулирует каждый из них

**Прим. от меня:** Представь у тебя на сайте фанпея есть несколько способов входа (ВК, Гугл, просто по логину и паролю), но нужно выбрать какой-то один, чтобы пользователи пользовались ток им. Ты как в случае с Observer создаешь просто несколько нужных классов с одними и теми же методами (типа register, login и т.п.), и потом уже добавляешь нужный в сервис авторизации. Он сам вызовет эти методы, а какой класс - ему похуй
```php 
<?php
interface SortStrategy {
    public function sort(array $data): array;
}

class BubbleSort implements SortStrategy {
    public function sort(array $data): array {
        // Реализация пузырьковой сортировки
        sort($data);
        return $data;
    }
}

class QuickSort implements SortStrategy {
    public function sort(array $data): array {
        // Реализация быстрой сортировки
        sort($data);
        return $data;
    }
}

class Sorter {
    private $strategy;

    public function __construct(SortStrategy $strategy) {
        $this->strategy = $strategy;
    }

    public function setStrategy(SortStrategy $strategy) {
        $this->strategy = $strategy;
    }

    public function sort(array $data): array {
        return $this->strategy->sort($data);
    }
}

// Использование
$data = [3, 1, 4, 1, 5, 9, 2, 6];

$sorter = new Sorter(new BubbleSort());
$result = $sorter->sort($data);

$sorter->setStrategy(new QuickSort());
$result = $sorter->sort($data);
?>
```