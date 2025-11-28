#### 1. Инкапсуляция - сокрытие внутренней реализации
```php
class BankAccount {
    private float $balance = 0; // Скрываем поле balance
    
    // Но даем безопасные методы для работы с ним
    public function deposit(float $amount): void {
        if ($amount > 0) $this->balance += $amount;
    }
    
    public function getBalance(): float {
        return $this->balance;
    }
}
```

#### 2. Наследование - расширение функциональности
```php
class Vehicle {
    public function move(): string { return "Moving"; }
}

// Переписываем (или дополняем) метод move из родительского класса
class Car extends Vehicle {
    public function move(): string { return "Driving"; }
}
```

#### 3. Полиморфизм - один интерфейс, разные реализации
У нас есть один интерфейс фигуры (Shape), но конкретные фигуры,
которые наследуются от Shape, вводят свой алгоритм расчета площади, который доступен
по одному и тому же методу area
```php
interface Shape {
    public function area(): float;
}

class Circle implements Shape {
    public function area(): float { return 3.14 * 5 * 5; }
}

class Square implements Shape {
    public function area(): float { return 10 * 10; }
}

function printArea(Shape $shape) {
    echo $shape->area(); // Работает с любым Shape
}
```

#### 4. Абстракция - работа на уровне концепций
Мы объявляем, что любой наследник класса Animal должен уметь издавать звуки (метод makeSound),
и каждый реализует его по своему
```php
abstract class Animal {
    abstract public function makeSound(): string;
    
    public function sleep(): string {
        return "Sleeping";
    }
}

class Cat extends Animal {
    public function makeSound(): string {
        return "Meow!";
    }
}
```
---
#### Чем отличается **интерфейс** от **абстракции**?

В примере выше абстрактный класс Animal вводил свой функционал (метод sleep),
помимо абстрактных методов. Интерфейс этого делать не может.

Суть в том, что абстрактный метод должен быть обязательно реализован во всех
дочерних классах, и интерфейс как бы просто описывает какие это должны быть методы,
он не вносит свой функционал.
---
#### Модификаторы доступа
- **public** - поле или метод будет доступен для всех
```php
class Cat {
    public string $name = "Cat";
}

new_cat = Cat();
echo $new_cat.name; // Имеем доступ к имени name откуда угодно
```
- **private** - поле или метод будет доступен только внутри класса
```php
class Cat {
    public string $name = "Cat";
    
    public function echo_name() {
        echo $this->name;
    }
}

new_cat = Cat();
new_cat->echo_name();   // Имеем доступ к публичной функции, которая сама поработает с приватным именем
echo $new_cat->name; // Ошибка, поле приватное
```
- **protected** - поле или метод будет доступен только внутри класса ИЛИ для прямого наследника
```php
class Animal {
    protected string $name = "Cat";
}

class Cat extends Animal {
    // здесь $name станет private, и для наследников класса Cat перестанет быть доступным
    public function echo_name() {
        echo $this->name;
    }
}

new_cat = Cat();
new_cat->echo_name();   // Имеем доступ к публичной функции, которая сама поработает с приватным именем
echo $new_cat->name; // Ошибка, поле приватное
```