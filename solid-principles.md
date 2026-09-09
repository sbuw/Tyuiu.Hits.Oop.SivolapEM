
# Часть 1. Экспресс-шпаргалка перед экзаменом

Краткая выжимка для быстрого повторения в день сдачи.

| Буква | Название принципа | Краткая суть простыми словами | Маркер ошибки в коде |
| --- | --- | --- | --- |
| **S** | **Single Responsibility** | Один класс = одна зона ответственности (одна причина для изменения). | Класс выполняет работу за разные модули (БД + UI + Логика). |
| **O** | **Open/Closed** | Расширяй систему новыми классами, не правь старый рабочий код. | Цепочки `if-else` или `switch` по типам данных. |
| **L** | **Liskov Substitution** | Наследник должен полностью соблюдать контракт предка и не ломать код. | `throw new NotImplementedException()` или падение программы. |
| **I** | **Interface Segregation** | Много маленьких специализированных интерфейсов лучше одного «толстого». | Пустые реализации методов или ошибки из-за ненужных методов. |
| **D** | **Dependency Inversion** | Завись от интерфейсов (абстракций), а не от конкретных классов. | Операторы `new` для внешних зависимостей внутри конструктора. |

---

# Часть 2. Полная практическая методичка по SOLID

---

## 1. S — Single Responsibility Principle (Принцип единственной обязанности)

**Определение:** Класс должен иметь только одну причину для изменения. Это значит, что он отвечает строго за одну задачу в системе.

**Ошибка:** Создание «классов-богов» (God Object), которые одновременно считают бизнес-логику, работают с базой данных и выводят информацию на экран.

#### ❌ Плохой код (Нарушение SRP)

Класс `Report` меняется и при изменении расчетов, и при смене формата консоли, и при смене файловой системы.

```csharp
public class Report
{
    public string Title { get; set; }

    public void GenerateReportData()
    {
        // Логика расчета отчета
    }

    public void PrintToConsole()
    {
        Console.WriteLine($"Отчет: {Title}");
    }

    public void SaveToFile(string path)
    {
        File.WriteAllText(path, Title);
    }
}

```

#### ✅ Правильный код (Соблюдение SRP)

Каждая обязанность вынесена в отдельный класс.

```csharp
// 1. Отвечает только за данные отчета
public class Report
{
    public string Title { get; set; }
}

// 2. Отвечает только за отображение
public class ReportPrinter
{
    public void Print(Report report)
    {
        Console.WriteLine($"Отчет: {report.Title}");
    }
}

// 3. Отвечает только за сохранение
public class ReportRepository
{
    public void SaveToFile(Report report, string path)
    {
        File.WriteAllText(path, report.Title);
    }
}

```

---

## 2. O — Open/Closed Principle (Принцип открытости / закрытости)

**Определение:** Программные сущности должны быть **открыты для расширения, но закрыты для модификации**. Новая функциональность добавляется путем создания новых классов, а не переписывания старого рабочего кода.

**Ошибка:** Использование цепочек `if-else` или `switch` для обработки типов. Чтобы добавить новый тип, приходится править готовый метод.

#### ❌ Плохой код (Нарушение OCP)

Класс `DiscountCalculator` приходится редактировать каждый раз, когда появляется новый тип скидки.

```csharp
public class DiscountCalculator
{
    public decimal CalculateDiscount(string customerType, decimal price)
    {
        if (customerType == "Regular")
            return price * 0.05m;
        else if (customerType == "VIP")
            return price * 0.20m;
        
        // Чтобы добавить "Partner", придется менять этот класс!
        return 0;
    }
}

```

#### ✅ Правильный код (Соблюдение OCP)

Логика вынесена за абстракцию. Для нового типа скидки создается **новый класс**.

```csharp
public interface IDiscountStrategy
{
    decimal Calculate(decimal price);
}

public class RegularDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal price) => price * 0.05m;
}

public class VipDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal price) => price * 0.20m;
}

// Добавляем новый функционал просто созданием нового файла:
public class PartnerDiscount : IDiscountStrategy
{
    public decimal Calculate(decimal price) => price * 0.30m;
}

// Этот класс навсегда закрыт от изменений
public class DiscountCalculator
{
    public decimal CalculateDiscount(IDiscountStrategy strategy, decimal price)
    {
        return strategy.Calculate(price);
    }
}

```

---

## 3. L — Liskov Substitution Principle (Принцип подстановки Барбары Лисков)

**Определение:** Объекты подтипов должны быть заменяемыми на экземпляры базовых типов **без нарушения корректности работы программы**. Наследник должен полностью выполнять контракт предка.

**Ошибка:** Наследник не может выполнить метод предка и выбрасывает исключение (`throw new NotImplementedException()`) или ломает логику программы.

#### ❌ Плохой код (Нарушение LSP)

Класс `Penguin` наследует `Bird`, но падает с ошибкой при попытке полета, ломая вызывающий код.

```csharp
public class Bird
{
    public virtual void Fly() => Console.WriteLine("Птица летит");
}

public class Penguin : Bird
{
    public override void Fly()
    {
        // Исключение обрушит программу в цикле foreach (Bird bird in birds)
        throw new NotSupportedException("Пингвины не умеют летать!");
    }
}

```

#### ✅ Правильный код (Соблюдение LSP)

Иерархия перестроена: летать умеют только те, кто реализует соответствующий контракт.

```csharp
public abstract class Bird
{
    public abstract void Eat();
}

public interface IFlyingBird
{
    void Fly();
}

public class Sparrow : Bird, IFlyingBird
{
    public override void Eat() => Console.WriteLine("Воробей клюет зерно");
    public void Fly() => Console.WriteLine("Воробей летит");
}

public class Penguin : Bird
{
    public override void Eat() => Console.WriteLine("Пингвин ест рыбу");
    // Пингвин не реализует IFlyingBird и никогда не сломает метод Fly()
}

```

---

## 4. I — Interface Segregation Principle (Принцип разделения интерфейсов)

**Определение:** Клиенты не должны зависеть от интерфейсов, которые они не используют. Множество узких интерфейсов лучше одного монолитного.

**Ошибка:** Создание гигантских («толстых») интерфейсов, которые заставляют классы делать методы-пустышки или выбрасывать ошибки.

#### ❌ Плохой код (Нарушение ISP)

Интерфейс `IWorker` заставляет `RobotWorker` реализовывать метод `Eat()`, который ему не нужен.

```csharp
public interface IWorker
{
    void Work();
    void Eat();
}

public class RobotWorker : IWorker
{
    public void Work() => Console.WriteLine("Робот работает");
    
    public void Eat() 
    {
        // Пустышка или ошибка, так как роботы не едят
        throw new NotImplementedException(); 
    }
}

```

#### ✅ Правильный код (Соблюдение ISP)

Большой интерфейс разбивается на ролевые интерфейсы.

```csharp
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public class HumanWorker : IWorkable, IEatable
{
    public void Work() => Console.WriteLine("Человек работает");
    public void Eat() => Console.WriteLine("Человек обедает");
}

public class RobotWorker : IWorkable
{
    public void Work() => Console.WriteLine("Робот работает");
}

```

---

## 5. D — Dependency Inversion Principle (Принцип инверсии зависимостей)

**Определение:** Модули верхнего уровня (бизнес-логика) и нижнего уровня (детали реализации) должны зависеть от **абстракций** (интерфейсов), а не друг от друга напрямую.

**Ошибка:** Создание жестких зависимостей через оператор `new` внутри конструктора класса бизнес-логики (Tight Coupling).

#### ❌ Плохой код (Нарушение DIP)

Класс `Car` намертво привязан к бензиновому двигателю. Невозможно подставить электродвигатель или написать Unit-тесты с заглушкой.

```csharp
public class PetrolEngine
{
    public void Start() => Console.WriteLine("Бензиновый двигатель запущен");
}

public class Car
{
    private PetrolEngine _engine;

    public Car()
    {
        // Жесткая связность!
        _engine = new PetrolEngine(); 
    }

    public void StartCar() => _engine.Start();
}

```

#### ✅ Правильный код (Соблюдение DIP)

Класс `Car` зависит от интерфейса `IEngine`. Экземпляр передается снаружи через конструктор (Dependency Injection).

```csharp
public interface IEngine
{
    void Start();
}

public class PetrolEngine : IEngine
{
    public void Start() => Console.WriteLine("Бензиновый двигатель запущен");
}

public class ElectricEngine : IEngine
{
    public void Start() => Console.WriteLine("Электродвигатель запущен");
}

public class Car
{
    private readonly IEngine _engine;

    // Внедрение зависимости (DI) через конструктор
    public Car(IEngine engine)
    {
        _engine = engine;
    }

    public void StartCar() => _engine.Start();
}

```