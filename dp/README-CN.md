## explain

1. [link](./README-EN.md)
2. 经过查看 state 相关
   - 暂时认为该笔记没有太大的价值
   - 其中的 `Real world example` + `In plain words` 可以看看(比较形象)

[toc]

## Introduction

1. 设计模式是对之前问题的总结而形成的, 指导如何处理具体的问题
2. 它不是具体的某个类, 每个包, 而是可重用的指导如何在指定场景下解决指定的问题的规则
3. 维基百科

   > In software engineering, a software design pattern is a general **reusable solution** to a commonly occurring problem within a given context in software design.
   > It is not a finished design that can be transformed directly into source or machine code.
   > It is a **description or template** for how to solve a problem that can be used in many different situations.

### ⚠️ Be Careful

1. 设计模式不是银弹
2. 在完全不了解的情况下, 不要使用: 只会增加代码混乱(dp 一般都是使用复杂度换扩展性)
3. 谨记设计模式是为了解决问题, 而不是找出一有问题

### Types of Design Patterns

1. [创建型](#creational-design-patterns)
2. [结构型](#structural-design-patterns)
3. [行为型](#behavioral-design-patterns)

## Creational Design Patterns

1. 关注点: 如何实例化(一个|一组)对象
2. Wikipedia says

   > In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.
   > The basic form of object creation could result in design problems or added complexity to the design.
   > Creational design patterns solve this problem by somehow controlling this object creation.

3. dps
   - [简单工厂](#-simple-factory)
   - [工厂方法](#-factory-method)
   - [抽象工厂](#-abstract-factory)
   - [构建者](#-builder)
   - [原型](#-prototype)
   - [单例](#-singleton)

### 🏠 Simple Factory

1.  Real world example

    - 需要门, 自己去制造一个门, 还是去让工厂制造好送过来

2.  In plain words

    - 简单工厂只是为客户端生成一个实例, 而不向客户端暴露任何实例化逻辑

3.  Wikipedia says

    > In object-oriented programming (OOP), a factory is an object for creating other objects
    > formally a factory is a function or method that returns objects of a varying prototype or class from some method call, which is assumed to be "new".

4.  **Programmatic Example**: 早门

    - First of all we have a door interface and the implementation

      ```php
      interface Door
      {
          public function getWidth(): float;
          public function getHeight(): float;
      }

      class WoodenDoor implements Door
      {
          protected $width;
          protected $height;

          public function __construct(float $width, float $height)
          {
              $this->width = $width;
              $this->height = $height;
          }

          public function getWidth(): float
          {
              return $this->width;
          }

          public function getHeight(): float
          {
              return $this->height;
          }
      }
      ```

    - Then we have our door factory that makes the door and returns it

      ```php
      class DoorFactory
      {
          public static function makeDoor($width, $height): Door
          {
              return new WoodenDoor($width, $height);
          }
      }
      ```

    - And then it can be used as

      ```php
      // Make me a door of 100x200
      $door = DoorFactory::makeDoor(100, 200);

      echo 'Width: ' . $door->getWidth();
      echo 'Height: ' . $door->getHeight();

      // Make me a door of 50x100
      $door2 = DoorFactory::makeDoor(50, 100);
      ```

5.  **When to Use?**

    - 当创建一个对象不仅仅是一些赋值并且涉及一些逻辑时, 将它放在一个专门的工厂而不是到处重复相同的代码是有意义的

### 🏭 Factory Method

1.  Real world example

    - ~~招聘情况: 不可能一个人面试一个职位, 根据职位空缺决定不同的经理面试不同的岗位人员~~

2.  In plain words

    - **提供了一种将实例化逻辑委托给子类的方法**

3.  Wikipedia says

    > In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created.
    > This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes, or implemented in a base class and optionally overridden by derived classes—rather than by calling a constructor.

4.  **Programmatic Example**

    - Taking our hiring manager example above. First of all we have an interviewer interface and some implementations for it

      ```php
      interface Interviewer
      {
          public function askQuestions();
      }

      class Developer implements Interviewer
      {
          public function askQuestions()
          {
              echo 'Asking about design patterns!';
          }
      }

      class CommunityExecutive implements Interviewer
      {
          public function askQuestions()
          {
              echo 'Asking about community building';
          }
      }
      ```

    - Now let us create our `HiringManager`

      ```php
      abstract class HiringManager
      {

          // Factory method
          abstract protected function makeInterviewer(): Interviewer;

          public function takeInterview()
          {
              $interviewer = $this->makeInterviewer();
              $interviewer->askQuestions();
          }
      }

      ```

    - Now any child can extend it and provide the required interviewer

      ```php
      class DevelopmentManager extends HiringManager
      {
          protected function makeInterviewer(): Interviewer
          {
              return new Developer();
          }
      }

      class MarketingManager extends HiringManager
      {
          protected function makeInterviewer(): Interviewer
          {
              return new CommunityExecutive();
          }
      }
      ```

    - and then it can be used as

      ```php
      $devManager = new DevelopmentManager();
      $devManager->takeInterview(); // Output: Asking about design patterns

      $marketingManager = new MarketingManager();
      $marketingManager->takeInterview(); // Output: Asking about community building.
      ```

5.  **When to use?**
    - 具体的创建逻辑不确定: 运行时决定由哪个子类创建(当客户不知道它可能需要什么确切的子类时)

### 🔨 Abstract Factory

1. Real world example

   - 需要门和安装师傅: 现在门之间存在依赖关系, 木门需要木匠, 铁门需要焊工等

2. In plain words

   - 工厂中的工厂: 一个工厂, 它将单独但相关/依赖的工厂组合在一起, 而不指定它们的具体类

3. Wikipedia says

   > The abstract factory pattern provides a way to encapsulate a group of individual factories that have a common theme without specifying their concrete classes

4. **Programmatic Example**

   - first of all we have our `Door` interface and some implementation for it

     ```php
     interface Door
     {
         public function getDescription();
     }

     class WoodenDoor implements Door
     {
         public function getDescription()
         {
             echo 'I am a wooden door';
         }
     }

     class IronDoor implements Door
     {
         public function getDescription()
         {
             echo 'I am an iron door';
         }
     }
     ```

   - Then we have some fitting experts for each door type

     ```php
     interface DoorFittingExpert
     {
         public function getDescription();
     }

     class Welder implements DoorFittingExpert
     {
         public function getDescription()
         {
             echo 'I can only fit iron doors';
         }
     }

     class Carpenter implements DoorFittingExpert
     {
         public function getDescription()
         {
             echo 'I can only fit wooden doors';
         }
     }
     ```

   - Now we have our abstract factory that would let us make family of related objects i.e. wooden door factory would create a wooden door and wooden door fitting expert and iron door factory would create an iron door and iron door fitting expert

     ```php
     interface DoorFactory
     {
         public function makeDoor(): Door;
         public function makeFittingExpert(): DoorFittingExpert;
     }

     // Wooden factory to return carpenter and wooden door
     class WoodenDoorFactory implements DoorFactory
     {
         public function makeDoor(): Door
         {
             return new WoodenDoor();
         }

         public function makeFittingExpert(): DoorFittingExpert
         {
             return new Carpenter();
         }
     }

     // Iron door factory to get iron door and the relevant fitting expert
     class IronDoorFactory implements DoorFactory
     {
         public function makeDoor(): Door
         {
             return new IronDoor();
         }

         public function makeFittingExpert(): DoorFittingExpert
         {
             return new Welder();
         }
     }
     ```

   - And then it can be used as

     ```php
     $woodenFactory = new WoodenDoorFactory();

     $door = $woodenFactory->makeDoor();
     $expert = $woodenFactory->makeFittingExpert();

     $door->getDescription();  // Output: I am a wooden door
     $expert->getDescription(); // Output: I can only fit wooden doors

     // Same for Iron Factory
     $ironFactory = new IronDoorFactory();

     $door = $ironFactory->makeDoor();
     $expert = $ironFactory->makeFittingExpert();

     $door->getDescription();  // Output: I am an iron door
     $expert->getDescription(); // Output: I can only fit iron doors
     ```

5. **When to use?**

   - 当存在相互关联的依赖关系并涉及不那么简单的创建逻辑时

### 👷 Builder

1. Real world example

   - 需要一个定制的门: 颜色, 大小, 烙印等指定特色, 每一步都能自己定制
   - 工厂模式: 从工厂获取, 一般不接收参数(定制), 直接获取完成后的对象

2. In plain words

   - 允许您创建不同风格(属性)的对象, 同时避免构造函数污染
   - 当创建一个对象涉及很多步骤和部件

3. Wikipedia says

   > 核心就是避免构造函数污染

4. **Programmatic Example**

   - 首先我们有我们想要制作的汉堡

     ```php
     class Burger
     {
         protected $size;

         protected $cheese = false;
         protected $pepperoni = false;
         protected $lettuce = false;
         protected $tomato = false;

         public function __construct(BurgerBuilder $builder)
         {
             $this->size = $builder->size;
             $this->cheese = $builder->cheese;
             $this->pepperoni = $builder->pepperoni;
             $this->lettuce = $builder->lettuce;
             $this->tomato = $builder->tomato;
         }
     }
     ```

   - And then we have the builder

     ```php
     class BurgerBuilder
     {
         public $size;

         public $cheese = false;
         public $pepperoni = false;
         public $lettuce = false;
         public $tomato = false;

         public function __construct(int $size)
         {
             $this->size = $size;
         }

         public function addPepperoni()
         {
             $this->pepperoni = true;
             return $this;
         }

         public function addLettuce()
         {
             $this->lettuce = true;
             return $this;
         }

         public function addCheese()
         {
             $this->cheese = true;
             return $this;
         }

         public function addTomato()
         {
             $this->tomato = true;
             return $this;
         }

         public function build(): Burger
         {
             return new Burger($this);
         }
     }
     ```

   - And then it can be used as:

     ```php
     $burger = (new BurgerBuilder(14))
                         ->addPepperoni()
                         ->addLettuce()
                         ->addTomato()
                         ->build();
     ```

5. **When to use?**

   - 当一个对象可能有多种风格并避免构造函数伸缩时
   - 与工厂模式的主要区别在于: 当创建是一步过程时使用工厂模式, 而当创建是多步过程时使用构建器模式

### 🐑 Prototype

1. Real world example

   - ~~Remember dolly? The sheep that was cloned! Lets not get into the details but the key point here is that it is all about cloning~~

2. In plain words

   - 通过克隆基于现有对象创建完全一样的副本对象, _之后可以修改各自对象_

3. Wikipedia says

   > The prototype pattern is a creational design pattern in software development.
   > It is used when the type of objects to create is determined by a prototypical instance, which is cloned to produce new objects.

4. **Programmatic Example**

   - In PHP, it can be easily done using `clone`

     ```php
     class Sheep
     {
         protected $name;
         protected $category;

         public function __construct(string $name, string $category = 'Mountain Sheep')
         {
             $this->name = $name;
             $this->category = $category;
         }

         public function setName(string $name)
         {
             $this->name = $name;
         }

         public function getName()
         {
             return $this->name;
         }

         public function setCategory(string $category)
         {
             $this->category = $category;
         }

         public function getCategory()
         {
             return $this->category;
         }
     }
     ```

   - Then it can be cloned like below

     ```php
     $original = new Sheep('Jolly');
     echo $original->getName(); // Jolly
     echo $original->getCategory(); // Mountain Sheep

     // Clone and modify what is required
     $cloned = clone $original;
     $cloned->setName('Dolly');
     echo $cloned->getName(); // Dolly
     echo $cloned->getCategory(); // Mountain sheep
     ```

5. **When to use?**

   - 当需要一个与现有对象相似的对象时
   - 或者当创建与克隆相比代价高昂时

### 💍 Singleton

1. Real world example

   - 一个国家只能有一位总统: 每当职责需要时, 必须让同一位总统采取行动

2. In plain words

   - 确保指定的类只会被创建一次

3. Wikipedia says

   > In software engineering, the singleton pattern is a software design pattern that restricts the instantiation of a class to one object.
   > This is useful when exactly one object is needed to coordinate actions across the system.

4. intros

   - 单例模式经常也会被认识为反模式的: 应该避免过度使用(合理使用即可)
   - 全局只有一个对象, 更改可能会影响未知的逻辑, 并且可能变得非常难以调试
   - 单例与代码强耦合
   - 测试困难: 模拟单例可能很困难

5. **Programmatic Example**

   - 将构造函数设为私有, 禁用克隆, 禁用扩展并创建一个静态变量来容纳实例

     ```php
     final class President
     {
         private static $instance;

         private function __construct()
         {
             // Hide the constructor
         }

         public static function getInstance(): President
         {
             if (!self::$instance) {
                 self::$instance = new self();
             }

             return self::$instance;
         }

         private function __clone()
         {
             // Disable cloning
         }

         private function __wakeup()
         {
             // Disable unserialize
         }
     }
     ```

   - usage

     ```php
     $president1 = President::getInstance();
     $president2 = President::getInstance();

     var_dump($president1 === $president2); // true
     ```

## Structural Design Patterns

1. 结构型模式关注点是对象组合(使用): 如果使用对象构建软件组件
2. Wikipedia says

   > In software engineering, structural design patterns are design patterns that ease the design by identifying a simple way to realize relationships between entities.

3. dps
   - [Adapter](#-adapter)
   - [Bridge](#-bridge)
   - [Composite](#-composite)
   - [Decorator](#-decorator)
   - [Facade](#-facade)
   - [Flyweight](#-flyweight)
   - [Proxy](#-proxy)

### 🔌 Adapter

Real world example

> Consider that you have some pictures in your memory card and you need to transfer them to your computer. In order to transfer them you need some kind of adapter that is compatible with your computer ports so that you can attach memory card to your computer. In this case card reader is an adapter.
> Another example would be the famous power adapter; a three legged plug can't be connected to a two pronged outlet, it needs to use a power adapter that makes it compatible with the two pronged outlet.
> Yet another example would be a translator translating words spoken by one person to another

In plain words

> Adapter pattern lets you wrap an otherwise incompatible object in an adapter to make it compatible with another class.

Wikipedia says

> In software engineering, the adapter pattern is a software design pattern that allows the interface of an existing class to be used as another interface. It is often used to make existing classes work with others without modifying their source code.

**Programmatic Example**

Consider a game where there is a hunter and he hunts lions.

First we have an interface `Lion` that all types of lions have to implement

```php
interface Lion
{
    public function roar();
}

class AfricanLion implements Lion
{
    public function roar()
    {
    }
}

class AsianLion implements Lion
{
    public function roar()
    {
    }
}
```

And hunter expects any implementation of `Lion` interface to hunt.

```php
class Hunter
{
    public function hunt(Lion $lion)
    {
        $lion->roar();
    }
}
```

Now let's say we have to add a `WildDog` in our game so that hunter can hunt that also. But we can't do that directly because dog has a different interface. To make it compatible for our hunter, we will have to create an adapter that is compatible

```php
// This needs to be added to the game
class WildDog
{
    public function bark()
    {
    }
}

// Adapter around wild dog to make it compatible with our game
class WildDogAdapter implements Lion
{
    protected $dog;

    public function __construct(WildDog $dog)
    {
        $this->dog = $dog;
    }

    public function roar()
    {
        $this->dog->bark();
    }
}
```

And now the `WildDog` can be used in our game using `WildDogAdapter`.

```php
$wildDog = new WildDog();
$wildDogAdapter = new WildDogAdapter($wildDog);

$hunter = new Hunter();
$hunter->hunt($wildDogAdapter);
```

### 🚡 Bridge

Real world example

> Consider you have a website with different pages and you are supposed to allow the user to change the theme. What would you do? Create multiple copies of each of the pages for each of the themes or would you just create separate theme and load them based on the user's preferences? Bridge pattern allows you to do the second i.e.

![With and without the bridge pattern](https://cloud.githubusercontent.com/assets/11269635/23065293/33b7aea0-f515-11e6-983f-98823c9845ee.png)

In Plain Words

> Bridge pattern is about preferring composition over inheritance. Implementation details are pushed from a hierarchy to another object with a separate hierarchy.

Wikipedia says

> The bridge pattern is a design pattern used in software engineering that is meant to "decouple an abstraction from its implementation so that the two can vary independently"

**Programmatic Example**

Translating our WebPage example from above. Here we have the `WebPage` hierarchy

```php
interface WebPage
{
    public function __construct(Theme $theme);
    public function getContent();
}

class About implements WebPage
{
    protected $theme;

    public function __construct(Theme $theme)
    {
        $this->theme = $theme;
    }

    public function getContent()
    {
        return "About page in " . $this->theme->getColor();
    }
}

class Careers implements WebPage
{
    protected $theme;

    public function __construct(Theme $theme)
    {
        $this->theme = $theme;
    }

    public function getContent()
    {
        return "Careers page in " . $this->theme->getColor();
    }
}
```

And the separate theme hierarchy

```php

interface Theme
{
    public function getColor();
}

class DarkTheme implements Theme
{
    public function getColor()
    {
        return 'Dark Black';
    }
}
class LightTheme implements Theme
{
    public function getColor()
    {
        return 'Off white';
    }
}
class AquaTheme implements Theme
{
    public function getColor()
    {
        return 'Light blue';
    }
}
```

And both the hierarchies

```php
$darkTheme = new DarkTheme();

$about = new About($darkTheme);
$careers = new Careers($darkTheme);

echo $about->getContent(); // "About page in Dark Black";
echo $careers->getContent(); // "Careers page in Dark Black";
```

### 🌿 Composite

Real world example

> Every organization is composed of employees. Each of the employees has the same features i.e. has a salary, has some responsibilities, may or may not report to someone, may or may not have some subordinates etc.

In plain words

> Composite pattern lets clients treat the individual objects in a uniform manner.

Wikipedia says

> In software engineering, the composite pattern is a partitioning design pattern. The composite pattern describes that a group of objects is to be treated in the same way as a single instance of an object. The intent of a composite is to "compose" objects into tree structures to represent part-whole hierarchies. Implementing the composite pattern lets clients treat individual objects and compositions uniformly.

**Programmatic Example**

Taking our employees example from above. Here we have different employee types

```php
interface Employee
{
    public function __construct(string $name, float $salary);
    public function getName(): string;
    public function setSalary(float $salary);
    public function getSalary(): float;
    public function getRoles(): array;
}

class Developer implements Employee
{
    protected $salary;
    protected $name;
    protected $roles;

    public function __construct(string $name, float $salary)
    {
        $this->name = $name;
        $this->salary = $salary;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function setSalary(float $salary)
    {
        $this->salary = $salary;
    }

    public function getSalary(): float
    {
        return $this->salary;
    }

    public function getRoles(): array
    {
        return $this->roles;
    }
}

class Designer implements Employee
{
    protected $salary;
    protected $name;
    protected $roles;

    public function __construct(string $name, float $salary)
    {
        $this->name = $name;
        $this->salary = $salary;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function setSalary(float $salary)
    {
        $this->salary = $salary;
    }

    public function getSalary(): float
    {
        return $this->salary;
    }

    public function getRoles(): array
    {
        return $this->roles;
    }
}
```

Then we have an organization which consists of several different types of employees

```php
class Organization
{
    protected $employees;

    public function addEmployee(Employee $employee)
    {
        $this->employees[] = $employee;
    }

    public function getNetSalaries(): float
    {
        $netSalary = 0;

        foreach ($this->employees as $employee) {
            $netSalary += $employee->getSalary();
        }

        return $netSalary;
    }
}
```

And then it can be used as

```php
// Prepare the employees
$john = new Developer('John Doe', 12000);
$jane = new Designer('Jane Doe', 15000);

// Add them to organization
$organization = new Organization();
$organization->addEmployee($john);
$organization->addEmployee($jane);

echo "Net salaries: " . $organization->getNetSalaries(); // Net Salaries: 27000
```

### ☕ Decorator

Real world example

> Imagine you run a car service shop offering multiple services. Now how do you calculate the bill to be charged? You pick one service and dynamically keep adding to it the prices for the provided services till you get the final cost. Here each type of service is a decorator.

In plain words

> Decorator pattern lets you dynamically change the behavior of an object at run time by wrapping them in an object of a decorator class.

Wikipedia says

> In object-oriented programming, the decorator pattern is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern is often useful for adhering to the Single Responsibility Principle, as it allows functionality to be divided between classes with unique areas of concern.

**Programmatic Example**

Lets take coffee for example. First of all we have a simple coffee implementing the coffee interface

```php
interface Coffee
{
    public function getCost();
    public function getDescription();
}

class SimpleCoffee implements Coffee
{
    public function getCost()
    {
        return 10;
    }

    public function getDescription()
    {
        return 'Simple coffee';
    }
}
```

We want to make the code extensible to allow options to modify it if required. Lets make some add-ons (decorators)

```php
class MilkCoffee implements Coffee
{
    protected $coffee;

    public function __construct(Coffee $coffee)
    {
        $this->coffee = $coffee;
    }

    public function getCost()
    {
        return $this->coffee->getCost() + 2;
    }

    public function getDescription()
    {
        return $this->coffee->getDescription() . ', milk';
    }
}

class WhipCoffee implements Coffee
{
    protected $coffee;

    public function __construct(Coffee $coffee)
    {
        $this->coffee = $coffee;
    }

    public function getCost()
    {
        return $this->coffee->getCost() + 5;
    }

    public function getDescription()
    {
        return $this->coffee->getDescription() . ', whip';
    }
}

class VanillaCoffee implements Coffee
{
    protected $coffee;

    public function __construct(Coffee $coffee)
    {
        $this->coffee = $coffee;
    }

    public function getCost()
    {
        return $this->coffee->getCost() + 3;
    }

    public function getDescription()
    {
        return $this->coffee->getDescription() . ', vanilla';
    }
}
```

Lets make a coffee now

```php
$someCoffee = new SimpleCoffee();
echo $someCoffee->getCost(); // 10
echo $someCoffee->getDescription(); // Simple Coffee

$someCoffee = new MilkCoffee($someCoffee);
echo $someCoffee->getCost(); // 12
echo $someCoffee->getDescription(); // Simple Coffee, milk

$someCoffee = new WhipCoffee($someCoffee);
echo $someCoffee->getCost(); // 17
echo $someCoffee->getDescription(); // Simple Coffee, milk, whip

$someCoffee = new VanillaCoffee($someCoffee);
echo $someCoffee->getCost(); // 20
echo $someCoffee->getDescription(); // Simple Coffee, milk, whip, vanilla
```

### 📦 Facade

Real world example

> How do you turn on the computer? "Hit the power button" you say! That is what you believe because you are using a simple interface that computer provides on the outside, internally it has to do a lot of stuff to make it happen. This simple interface to the complex subsystem is a facade.

In plain words

> Facade pattern provides a simplified interface to a complex subsystem.

Wikipedia says

> A facade is an object that provides a simplified interface to a larger body of code, such as a class library.

**Programmatic Example**

Taking our computer example from above. Here we have the computer class

```php
class Computer
{
    public function getElectricShock()
    {
        echo "Ouch!";
    }

    public function makeSound()
    {
        echo "Beep beep!";
    }

    public function showLoadingScreen()
    {
        echo "Loading..";
    }

    public function bam()
    {
        echo "Ready to be used!";
    }

    public function closeEverything()
    {
        echo "Bup bup bup buzzzz!";
    }

    public function sooth()
    {
        echo "Zzzzz";
    }

    public function pullCurrent()
    {
        echo "Haaah!";
    }
}
```

Here we have the facade

```php
class ComputerFacade
{
    protected $computer;

    public function __construct(Computer $computer)
    {
        $this->computer = $computer;
    }

    public function turnOn()
    {
        $this->computer->getElectricShock();
        $this->computer->makeSound();
        $this->computer->showLoadingScreen();
        $this->computer->bam();
    }

    public function turnOff()
    {
        $this->computer->closeEverything();
        $this->computer->pullCurrent();
        $this->computer->sooth();
    }
}
```

Now to use the facade

```php
$computer = new ComputerFacade(new Computer());
$computer->turnOn(); // Ouch! Beep beep! Loading.. Ready to be used!
$computer->turnOff(); // Bup bup buzzz! Haah! Zzzzz
```

### 🍃 Flyweight

Real world example

> Did you ever have fresh tea from some stall? They often make more than one cup that you demanded and save the rest for any other customer so to save the resources e.g. gas etc. Flyweight pattern is all about that i.e. sharing.

In plain words

> It is used to minimize memory usage or computational expenses by sharing as much as possible with similar objects.

Wikipedia says

> In computer programming, flyweight is a software design pattern. A flyweight is an object that minimizes memory use by sharing as much data as possible with other similar objects; it is a way to use objects in large numbers when a simple repeated representation would use an unacceptable amount of memory.

**Programmatic example**

Translating our tea example from above. First of all we have tea types and tea maker

```php
// Anything that will be cached is flyweight.
// Types of tea here will be flyweights.
class KarakTea
{
}

// Acts as a factory and saves the tea
class TeaMaker
{
    protected $availableTea = [];

    public function make($preference)
    {
        if (empty($this->availableTea[$preference])) {
            $this->availableTea[$preference] = new KarakTea();
        }

        return $this->availableTea[$preference];
    }
}
```

Then we have the `TeaShop` which takes orders and serves them

```php
class TeaShop
{
    protected $orders;
    protected $teaMaker;

    public function __construct(TeaMaker $teaMaker)
    {
        $this->teaMaker = $teaMaker;
    }

    public function takeOrder(string $teaType, int $table)
    {
        $this->orders[$table] = $this->teaMaker->make($teaType);
    }

    public function serve()
    {
        foreach ($this->orders as $table => $tea) {
            echo "Serving tea to table# " . $table;
        }
    }
}
```

And it can be used as below

```php
$teaMaker = new TeaMaker();
$shop = new TeaShop($teaMaker);

$shop->takeOrder('less sugar', 1);
$shop->takeOrder('more milk', 2);
$shop->takeOrder('without sugar', 5);

$shop->serve();
// Serving tea to table# 1
// Serving tea to table# 2
// Serving tea to table# 5
```

### 🎱 Proxy

1. Real world example

   - ~~门的主要功能是打开，但在它上面添加了一个代理来添加一些功能~~

2. In plain words

   - 使用代理模式, 一个类代表另一个类的功能, 进行访问、控制或扩展

3. Wikipedia says

   > A proxy, in its most general form, is a class functioning as an interface to something else.
   > A proxy is a wrapper or agent object that is being called by the client to access the real serving object behind the scenes.
   > Use of the proxy can simply be forwarding to the real object, or can provide additional logic.
   > In the proxy extra functionality can be provided, for example caching when operations on the real object are resource intensive, or checking preconditions before operations on the real object are invoked.

4. **Programmatic Example**

   - Taking our security door example from above. Firstly we have the door interface and an implementation of door

     ```php
     interface Door
     {
         public function open();
         public function close();
     }

     class LabDoor implements Door
     {
         public function open()
         {
             echo "Opening lab door";
         }

         public function close()
         {
             echo "Closing the lab door";
         }
     }
     ```

   - Then we have a proxy to secure any doors that we want

     ```php
     class SecuredDoor
     {
         protected $door;

         public function __construct(Door $door)
         {
             $this->door = $door;
         }

         public function open($password)
         {
             if ($this->authenticate($password)) {
                 $this->door->open();
             } else {
                 echo "Big no! It ain't possible.";
             }
         }

         public function authenticate($password)
         {
             return $password === '$ecr@t';
         }

         public function close()
         {
             $this->door->close();
         }
     }
     ```

   - And here is how it can be used

     ```php
     $door = new SecuredDoor(new LabDoor());
     $door->open('invalid'); // Big no! It ain't possible.

     $door->open('$ecr@t'); // Opening lab door
     $door->close(); // Closing lab door
     ```

## Behavioral Design Patterns

1. **关注对象之间的职责分配**: 如何在软件组件中**运行一个行为**
2. 与结构模式的不同

   - 不仅指定了结构
   - 而且还概述了它们之间消息传递/通信的模式

3. Wikipedia says

   > In software engineering, behavioral design patterns are design patterns that identify common communication patterns between objects and realize these patterns.
   > By doing so, these patterns increase flexibility in carrying out this communication.

4. dps
   - [👽 Mediator](#-mediator)
   - [💾 Memento](#-memento)
   - [😎 Observer](#-observer)
   - [🏃 Visitor](#-visitor)
   - [💡 Strategy](#-strategy)
   - [💢 State](#-state)
   - [📒 Template Method](#-template-method)
   - [🚦 Wrap Up Folks](#-wrap-up-folks)
   - [👬 Contribution](#-contribution)

### 🔗 Chain of Responsibility

Real world example

> For example, you have three payment methods (`A`, `B` and `C`) setup in your account; each having a different amount in it. `A` has 100 USD, `B` has 300 USD and `C` having 1000 USD and the preference for payments is chosen as `A` then `B` then `C`. You try to purchase something that is worth 210 USD. Using Chain of Responsibility, first of all account `A` will be checked if it can make the purchase, if yes purchase will be made and the chain will be broken. If not, request will move forward to account `B` checking for amount if yes chain will be broken otherwise the request will keep forwarding till it finds the suitable handler. Here `A`, `B` and `C` are links of the chain and the whole phenomenon is Chain of Responsibility.

In plain words

> It helps building a chain of objects. Request enters from one end and keeps going from object to object till it finds the suitable handler.

Wikipedia says

> In object-oriented design, the chain-of-responsibility pattern is a design pattern consisting of a source of command objects and a series of processing objects. Each processing object contains logic that defines the types of command objects that it can handle; the rest are passed to the next processing object in the chain.

**Programmatic Example**

Translating our account example above. First of all we have a base account having the logic for chaining the accounts together and some accounts

```php
abstract class Account
{
    protected $successor;
    protected $balance;

    public function setNext(Account $account)
    {
        $this->successor = $account;
    }

    public function pay(float $amountToPay)
    {
        if ($this->canPay($amountToPay)) {
            echo sprintf('Paid %s using %s' . PHP_EOL, $amountToPay, get_called_class());
        } elseif ($this->successor) {
            echo sprintf('Cannot pay using %s. Proceeding ..' . PHP_EOL, get_called_class());
            $this->successor->pay($amountToPay);
        } else {
            throw new Exception('None of the accounts have enough balance');
        }
    }

    public function canPay($amount): bool
    {
        return $this->balance >= $amount;
    }
}

class Bank extends Account
{
    protected $balance;

    public function __construct(float $balance)
    {
        $this->balance = $balance;
    }
}

class Paypal extends Account
{
    protected $balance;

    public function __construct(float $balance)
    {
        $this->balance = $balance;
    }
}

class Bitcoin extends Account
{
    protected $balance;

    public function __construct(float $balance)
    {
        $this->balance = $balance;
    }
}
```

Now let's prepare the chain using the links defined above (i.e. Bank, Paypal, Bitcoin)

```php
// Let's prepare a chain like below
//      $bank->$paypal->$bitcoin
//
// First priority bank
//      If bank can't pay then paypal
//      If paypal can't pay then bit coin

$bank = new Bank(100);          // Bank with balance 100
$paypal = new Paypal(200);      // Paypal with balance 200
$bitcoin = new Bitcoin(300);    // Bitcoin with balance 300

$bank->setNext($paypal);
$paypal->setNext($bitcoin);

// Let's try to pay using the first priority i.e. bank
$bank->pay(259);

// Output will be
// ==============
// Cannot pay using bank. Proceeding ..
// Cannot pay using paypal. Proceeding ..:
// Paid 259 using Bitcoin!
```

### 👮 Command

Real world example

> A generic example would be you ordering food at a restaurant. You (i.e. `Client`) ask the waiter (i.e. `Invoker`) to bring some food (i.e. `Command`) and waiter simply forwards the request to Chef (i.e. `Receiver`) who has the knowledge of what and how to cook.
> Another example would be you (i.e. `Client`) switching on (i.e. `Command`) the television (i.e. `Receiver`) using a remote control (`Invoker`).

In plain words

> Allows you to encapsulate actions in objects. The key idea behind this pattern is to provide the means to decouple client from receiver.

Wikipedia says

> In object-oriented programming, the command pattern is a behavioral design pattern in which an object is used to encapsulate all information needed to perform an action or trigger an event at a later time. This information includes the method name, the object that owns the method and values for the method parameters.

**Programmatic Example**

First of all we have the receiver that has the implementation of every action that could be performed

```php
// Receiver
class Bulb
{
    public function turnOn()
    {
        echo "Bulb has been lit";
    }

    public function turnOff()
    {
        echo "Darkness!";
    }
}
```

then we have an interface that each of the commands are going to implement and then we have a set of commands

```php
interface Command
{
    public function execute();
    public function undo();
    public function redo();
}

// Command
class TurnOn implements Command
{
    protected $bulb;

    public function __construct(Bulb $bulb)
    {
        $this->bulb = $bulb;
    }

    public function execute()
    {
        $this->bulb->turnOn();
    }

    public function undo()
    {
        $this->bulb->turnOff();
    }

    public function redo()
    {
        $this->execute();
    }
}

class TurnOff implements Command
{
    protected $bulb;

    public function __construct(Bulb $bulb)
    {
        $this->bulb = $bulb;
    }

    public function execute()
    {
        $this->bulb->turnOff();
    }

    public function undo()
    {
        $this->bulb->turnOn();
    }

    public function redo()
    {
        $this->execute();
    }
}
```

Then we have an `Invoker` with whom the client will interact to process any commands

```php
// Invoker
class RemoteControl
{
    public function submit(Command $command)
    {
        $command->execute();
    }
}
```

Finally let's see how we can use it in our client

```php
$bulb = new Bulb();

$turnOn = new TurnOn($bulb);
$turnOff = new TurnOff($bulb);

$remote = new RemoteControl();
$remote->submit($turnOn); // Bulb has been lit!
$remote->submit($turnOff); // Darkness!
```

Command pattern can also be used to implement a transaction based system. Where you keep maintaining the history of commands as soon as you execute them. If the final command is successfully executed, all good otherwise just iterate through the history and keep executing the `undo` on all the executed commands.

### ➿ Iterator

Real world example

> An old radio set will be a good example of iterator, where user could start at some channel and then use next or previous buttons to go through the respective channels. Or take an example of MP3 player or a TV set where you could press the next and previous buttons to go through the consecutive channels or in other words they all provide an interface to iterate through the respective channels, songs or radio stations.

In plain words

> It presents a way to access the elements of an object without exposing the underlying presentation.

Wikipedia says

> In object-oriented programming, the iterator pattern is a design pattern in which an iterator is used to traverse a container and access the container's elements. The iterator pattern decouples algorithms from containers; in some cases, algorithms are necessarily container-specific and thus cannot be decoupled.

**Programmatic example**

In PHP it is quite easy to implement using SPL (Standard PHP Library). Translating our radio stations example from above. First of all we have `RadioStation`

```php
class RadioStation
{
    protected $frequency;

    public function __construct(float $frequency)
    {
        $this->frequency = $frequency;
    }

    public function getFrequency(): float
    {
        return $this->frequency;
    }
}
```

Then we have our iterator

```php
use Countable;
use Iterator;

class StationList implements Countable, Iterator
{
    /** @var RadioStation[] $stations */
    protected $stations = [];

    /** @var int $counter */
    protected $counter;

    public function addStation(RadioStation $station)
    {
        $this->stations[] = $station;
    }

    public function removeStation(RadioStation $toRemove)
    {
        $toRemoveFrequency = $toRemove->getFrequency();
        $this->stations = array_filter($this->stations, function (RadioStation $station) use ($toRemoveFrequency) {
            return $station->getFrequency() !== $toRemoveFrequency;
        });
    }

    public function count(): int
    {
        return count($this->stations);
    }

    public function current(): RadioStation
    {
        return $this->stations[$this->counter];
    }

    public function key()
    {
        return $this->counter;
    }

    public function next()
    {
        $this->counter++;
    }

    public function rewind()
    {
        $this->counter = 0;
    }

    public function valid(): bool
    {
        return isset($this->stations[$this->counter]);
    }
}
```

And then it can be used as

```php
$stationList = new StationList();

$stationList->addStation(new RadioStation(89));
$stationList->addStation(new RadioStation(101));
$stationList->addStation(new RadioStation(102));
$stationList->addStation(new RadioStation(103.2));

foreach($stationList as $station) {
    echo $station->getFrequency() . PHP_EOL;
}

$stationList->removeStation(new RadioStation(89)); // Will remove station 89
```

### 👽 Mediator

Real world example

> A general example would be when you talk to someone on your mobile phone, there is a network provider sitting between you and them and your conversation goes through it instead of being directly sent. In this case network provider is mediator.

In plain words

> Mediator pattern adds a third party object (called mediator) to control the interaction between two objects (called colleagues). It helps reduce the coupling between the classes communicating with each other. Because now they don't need to have the knowledge of each other's implementation.

Wikipedia says

> In software engineering, the mediator pattern defines an object that encapsulates how a set of objects interact. This pattern is considered to be a behavioral pattern due to the way it can alter the program's running behavior.

**Programmatic Example**

Here is the simplest example of a chat room (i.e. mediator) with users (i.e. colleagues) sending messages to each other.

First of all, we have the mediator i.e. the chat room

```php
interface ChatRoomMediator
{
    public function showMessage(User $user, string $message);
}

// Mediator
class ChatRoom implements ChatRoomMediator
{
    public function showMessage(User $user, string $message)
    {
        $time = date('M d, y H:i');
        $sender = $user->getName();

        echo $time . '[' . $sender . ']:' . $message;
    }
}
```

Then we have our users i.e. colleagues

```php
class User {
    protected $name;
    protected $chatMediator;

    public function __construct(string $name, ChatRoomMediator $chatMediator) {
        $this->name = $name;
        $this->chatMediator = $chatMediator;
    }

    public function getName() {
        return $this->name;
    }

    public function send($message) {
        $this->chatMediator->showMessage($this, $message);
    }
}
```

And the usage

```php
$mediator = new ChatRoom();

$john = new User('John Doe', $mediator);
$jane = new User('Jane Doe', $mediator);

$john->send('Hi there!');
$jane->send('Hey!');

// Output will be
// Feb 14, 10:58 [John]: Hi there!
// Feb 14, 10:58 [Jane]: Hey!
```

### 💾 Memento

Real world example

> Take the example of calculator (i.e. originator), where whenever you perform some calculation the last calculation is saved in memory (i.e. memento) so that you can get back to it and maybe get it restored using some action buttons (i.e. caretaker).

In plain words

> Memento pattern is about capturing and storing the current state of an object in a manner that it can be restored later on in a smooth manner.

Wikipedia says

> The memento pattern is a software design pattern that provides the ability to restore an object to its previous state (undo via rollback).

Usually useful when you need to provide some sort of undo functionality.

**Programmatic Example**

Lets take an example of text editor which keeps saving the state from time to time and that you can restore if you want.

First of all we have our memento object that will be able to hold the editor state

```php
class EditorMemento
{
    protected $content;

    public function __construct(string $content)
    {
        $this->content = $content;
    }

    public function getContent()
    {
        return $this->content;
    }
}
```

Then we have our editor i.e. originator that is going to use memento object

```php
class Editor
{
    protected $content = '';

    public function type(string $words)
    {
        $this->content = $this->content . ' ' . $words;
    }

    public function getContent()
    {
        return $this->content;
    }

    public function save()
    {
        return new EditorMemento($this->content);
    }

    public function restore(EditorMemento $memento)
    {
        $this->content = $memento->getContent();
    }
}
```

And then it can be used as

```php
$editor = new Editor();

// Type some stuff
$editor->type('This is the first sentence.');
$editor->type('This is second.');

// Save the state to restore to : This is the first sentence. This is second.
$saved = $editor->save();

// Type some more
$editor->type('And this is third.');

// Output: Content before Saving
echo $editor->getContent(); // This is the first sentence. This is second. And this is third.

// Restoring to last saved state
$editor->restore($saved);

$editor->getContent(); // This is the first sentence. This is second.
```

### 😎 Observer

Real world example

> A good example would be the job seekers where they subscribe to some job posting site and they are notified whenever there is a matching job opportunity.

In plain words

> Defines a dependency between objects so that whenever an object changes its state, all its dependents are notified.

Wikipedia says

> The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.

**Programmatic example**

Translating our example from above. First of all we have job seekers that need to be notified for a job posting

```php
class JobPost
{
    protected $title;

    public function __construct(string $title)
    {
        $this->title = $title;
    }

    public function getTitle()
    {
        return $this->title;
    }
}

class JobSeeker implements Observer
{
    protected $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function onJobPosted(JobPost $job)
    {
        // Do something with the job posting
        echo 'Hi ' . $this->name . '! New job posted: '. $job->getTitle();
    }
}
```

Then we have our job postings to which the job seekers will subscribe

```php
class EmploymentAgency implements Observable
{
    protected $observers = [];

    protected function notify(JobPost $jobPosting)
    {
        foreach ($this->observers as $observer) {
            $observer->onJobPosted($jobPosting);
        }
    }

    public function attach(Observer $observer)
    {
        $this->observers[] = $observer;
    }

    public function addJob(JobPost $jobPosting)
    {
        $this->notify($jobPosting);
    }
}
```

Then it can be used as

```php
// Create subscribers
$johnDoe = new JobSeeker('John Doe');
$janeDoe = new JobSeeker('Jane Doe');

// Create publisher and attach subscribers
$jobPostings = new EmploymentAgency();
$jobPostings->attach($johnDoe);
$jobPostings->attach($janeDoe);

// Add a new job and see if subscribers get notified
$jobPostings->addJob(new JobPost('Software Engineer'));

// Output
// Hi John Doe! New job posted: Software Engineer
// Hi Jane Doe! New job posted: Software Engineer
```

### 🏃 Visitor

Real world example

> Consider someone visiting Dubai. They just need a way (i.e. visa) to enter Dubai. After arrival, they can come and visit any place in Dubai on their own without having to ask for permission or to do some leg work in order to visit any place here; just let them know of a place and they can visit it. Visitor pattern lets you do just that, it helps you add places to visit so that they can visit as much as they can without having to do any legwork.

In plain words

> Visitor pattern lets you add further operations to objects without having to modify them.

Wikipedia says

> In object-oriented programming and software engineering, the visitor design pattern is a way of separating an algorithm from an object structure on which it operates. A practical result of this separation is the ability to add new operations to existing object structures without modifying those structures. It is one way to follow the open/closed principle.

**Programmatic example**

Let's take an example of a zoo simulation where we have several different kinds of animals and we have to make them Sound. Let's translate this using visitor pattern

```php
// Visitee
interface Animal
{
    public function accept(AnimalOperation $operation);
}

// Visitor
interface AnimalOperation
{
    public function visitMonkey(Monkey $monkey);
    public function visitLion(Lion $lion);
    public function visitDolphin(Dolphin $dolphin);
}
```

Then we have our implementations for the animals

```php
class Monkey implements Animal
{
    public function shout()
    {
        echo 'Ooh oo aa aa!';
    }

    public function accept(AnimalOperation $operation)
    {
        $operation->visitMonkey($this);
    }
}

class Lion implements Animal
{
    public function roar()
    {
        echo 'Roaaar!';
    }

    public function accept(AnimalOperation $operation)
    {
        $operation->visitLion($this);
    }
}

class Dolphin implements Animal
{
    public function speak()
    {
        echo 'Tuut tuttu tuutt!';
    }

    public function accept(AnimalOperation $operation)
    {
        $operation->visitDolphin($this);
    }
}
```

Let's implement our visitor

```php
class Speak implements AnimalOperation
{
    public function visitMonkey(Monkey $monkey)
    {
        $monkey->shout();
    }

    public function visitLion(Lion $lion)
    {
        $lion->roar();
    }

    public function visitDolphin(Dolphin $dolphin)
    {
        $dolphin->speak();
    }
}
```

And then it can be used as

```php
$monkey = new Monkey();
$lion = new Lion();
$dolphin = new Dolphin();

$speak = new Speak();

$monkey->accept($speak);    // Ooh oo aa aa!
$lion->accept($speak);      // Roaaar!
$dolphin->accept($speak);   // Tuut tutt tuutt!
```

We could have done this simply by having an inheritance hierarchy for the animals but then we would have to modify the animals whenever we would have to add new actions to animals. But now we will not have to change them. For example, let's say we are asked to add the jump behavior to the animals, we can simply add that by creating a new visitor i.e.

```php
class Jump implements AnimalOperation
{
    public function visitMonkey(Monkey $monkey)
    {
        echo 'Jumped 20 feet high! on to the tree!';
    }

    public function visitLion(Lion $lion)
    {
        echo 'Jumped 7 feet! Back on the ground!';
    }

    public function visitDolphin(Dolphin $dolphin)
    {
        echo 'Walked on water a little and disappeared';
    }
}
```

And for the usage

```php
$jump = new Jump();

$monkey->accept($speak);   // Ooh oo aa aa!
$monkey->accept($jump);    // Jumped 20 feet high! on to the tree!

$lion->accept($speak);     // Roaaar!
$lion->accept($jump);      // Jumped 7 feet! Back on the ground!

$dolphin->accept($speak);  // Tuut tutt tuutt!
$dolphin->accept($jump);   // Walked on water a little and disappeared
```

### 💡 Strategy

Real world example

> Consider the example of sorting, we implemented bubble sort but the data started to grow and bubble sort started getting very slow. In order to tackle this we implemented Quick sort. But now although the quick sort algorithm was doing better for large datasets, it was very slow for smaller datasets. In order to handle this we implemented a strategy where for small datasets, bubble sort will be used and for larger, quick sort.

In plain words

> Strategy pattern allows you to switch the algorithm or strategy based upon the situation.

Wikipedia says

> In computer programming, the strategy pattern (also known as the policy pattern) is a behavioural software design pattern that enables an algorithm's behavior to be selected at runtime.

**Programmatic example**

Translating our example from above. First of all we have our strategy interface and different strategy implementations

```php
interface SortStrategy
{
    public function sort(array $dataset): array;
}

class BubbleSortStrategy implements SortStrategy
{
    public function sort(array $dataset): array
    {
        echo "Sorting using bubble sort";

        // Do sorting
        return $dataset;
    }
}

class QuickSortStrategy implements SortStrategy
{
    public function sort(array $dataset): array
    {
        echo "Sorting using quick sort";

        // Do sorting
        return $dataset;
    }
}
```

And then we have our client that is going to use any strategy

```php
class Sorter
{
    protected $sorter;

    public function __construct(SortStrategy $sorter)
    {
        $this->sorter = $sorter;
    }

    public function sort(array $dataset): array
    {
        return $this->sorter->sort($dataset);
    }
}
```

And it can be used as

```php
$dataset = [1, 5, 4, 3, 2, 8];

$sorter = new Sorter(new BubbleSortStrategy());
$sorter->sort($dataset); // Output : Sorting using bubble sort

$sorter = new Sorter(new QuickSortStrategy());
$sorter->sort($dataset); // Output : Sorting using quick sort
```

### 💢 State

1.  物理举例

    - 想象一下, 您正在使用一些绘图应用程序, 您选择画笔进行绘图
    - 现在画笔根据所选颜色更改其行为
    - 即如果您选择红色, 它将绘制为红色, 如果选择蓝色, 则它将绘制为蓝色等

2.  核心

    - 它允许在状态更改时更改类的行为

3.  Wikipedia says

    > The state pattern is a behavioral software design pattern that implements a state machine in an **object-oriented** way.
    > With the state pattern, a state machine is implemented by implementing each individual state as a derived class of the state pattern interface, and implementing state transitions by invoking methods defined by the pattern's superclass.

4.  状态模式和策略模式有一定的相同性

5.  **实例代码**: text editor

    - 如果您选择粗体, 它开始以粗体书写;
    - 如果选择斜体, 则以斜体等;

    - interface & implementations

      ```php
      interface WritingState
      {
          public function write(string $words);
      }

      class UpperCase implements WritingState
      {
          public function write(string $words)
          {
              echo strtoupper($words);
          }
      }

      class LowerCase implements WritingState
      {
          public function write(string $words)
          {
              echo strtolower($words);
          }
      }

      class DefaultText implements WritingState
      {
          public function write(string $words)
          {
              echo $words;
          }
      }
      ```

    - Then we have our editor

      ```php
      class TextEditor
      {
          protected $state;

          public function __construct(WritingState $state)
          {
              $this->state = $state;
          }

          public function setState(WritingState $state)
          {
              $this->state = $state;
          }

          public function type(string $words)
          {
              $this->state->write($words);
          }
      }
      ```

    - And then it can be used as

      ```php
      $editor = new TextEditor(new DefaultText());

      $editor->type('First line');

      $editor->setState(new UpperCase());

      $editor->type('Second line');
      $editor->type('Third line');

      $editor->setState(new LowerCase());

      $editor->type('Fourth line');
      $editor->type('Fifth line');

      // Output:
      // First line
      // SECOND LINE
      // THIRD LINE
      // fourth line
      // fifth line
      ```

### 📒 Template Method

Real world example

> Suppose we are getting some house built. The steps for building might look like
>
> - Prepare the base of house
> - Build the walls
> - Add roof
> - Add other floors

> The order of these steps could never be changed i.e. you can't build the roof before building the walls etc but each of the steps could be modified for example walls can be made of wood or polyester or stone.

In plain words

> Template method defines the skeleton of how a certain algorithm could be performed, but defers the implementation of those steps to the children classes.

Wikipedia says

> In software engineering, the template method pattern is a behavioral design pattern that defines the program skeleton of an algorithm in an operation, deferring some steps to subclasses. It lets one redefine certain steps of an algorithm without changing the algorithm's structure.

**Programmatic Example**

Imagine we have a build tool that helps us test, lint, build, generate build reports (i.e. code coverage reports, linting report etc) and deploy our app on the test server.

First of all we have our base class that specifies the skeleton for the build algorithm

```php
abstract class Builder
{

    // Template method
    final public function build()
    {
        $this->test();
        $this->lint();
        $this->assemble();
        $this->deploy();
    }

    abstract public function test();
    abstract public function lint();
    abstract public function assemble();
    abstract public function deploy();
}
```

Then we can have our implementations

```php
class AndroidBuilder extends Builder
{
    public function test()
    {
        echo 'Running android tests';
    }

    public function lint()
    {
        echo 'Linting the android code';
    }

    public function assemble()
    {
        echo 'Assembling the android build';
    }

    public function deploy()
    {
        echo 'Deploying android build to server';
    }
}

class IosBuilder extends Builder
{
    public function test()
    {
        echo 'Running ios tests';
    }

    public function lint()
    {
        echo 'Linting the ios code';
    }

    public function assemble()
    {
        echo 'Assembling the ios build';
    }

    public function deploy()
    {
        echo 'Deploying ios build to server';
    }
}
```

And then it can be used as

```php
$androidBuilder = new AndroidBuilder();
$androidBuilder->build();

// Output:
// Running android tests
// Linting the android code
// Assembling the android build
// Deploying android build to server

$iosBuilder = new IosBuilder();
$iosBuilder->build();

// Output:
// Running ios tests
// Linting the ios code
// Assembling the ios build
// Deploying ios build to server
```
