---
tags:
  - этап1
  - todo
---
Можно использовать модификатор `record` для объявления типа, который предоставляет встроенный функционал для сравнения типов и печати в строку. `record class` синонимичен `record` и создаёт [ссылочный тип](Reference%20типы), `record struct` создаёт [значимый тип](Value%20типы).

При объявлении первичного конструктора на `record` компилятор генерирует публичные поля () для аргументов первичного конструктора.

```cs
public record Person(string FirstName, string LastName);
```

```cs
public record Person
{
    public required string FirstName { get; init; }
    public required string LastName { get; init; }
};
```

```cs
public readonly record struct Point(double X, double Y, double Z);
```

```cs
public record struct Point
{
    public double X { get; init; }
    public double Y { get; init; }
    public double Z { get; init; }
}
```

Так же можно создавать рекорды с мутабельными полями и свойствами:

```cs
public record Person
{
    public required string FirstName { get; set; }
    public required string LastName { get; set; }
};
```

Хотя рекорды могут быть мутабельными, они в первую очередь предназначены для работы с иммутабельными моделями данных. Рекорды предлагают следующий функционал:

# Позиционный синтаксис для объявления свойств и полей

```cs
public record Person(string FirstName, string LastName);

public static void Main()
{
    Person person = new("Nancy", "Davolio");
    Console.WriteLine(person);
    // output: Person { FirstName = Nancy, LastName = Davolio }
}
```


При использовании позиционного синтаксиса компилятором создаются:
- Автоматические свойства
	- Для `record`и `readonly record struct` типов создаются init-only свойства
	- Для `record struct` типов создаются read-write свойства
- Первичный конструктор, параметры которого совпадают с позиционными параметрами объявления рекорда
- Конструктор по умолчанию без аргументов, который обнуляет все поля для `record struct`
- Метод `Deconstruct` с [out](in,%20ref,%20out) параметрами для каждого позиционного параметра. Метод разбирает только позиционные свойства, игнорируя объявленные свойства стандартным способом