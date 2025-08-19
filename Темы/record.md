---
tags:
  - этап1
  - done
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
- Метод `Deconstruct` с [out](Параметры%20методов%20и%20модификаторы.md) параметрами для каждого позиционного параметра. Метод разбирает только позиционные свойства, игнорируя объявленные свойства стандартным способом
К любому из элементов, создаваемому компилятором из объявления рекорда можно добавить атрибут, указав таргет, к которому применить этот атрибут (`property`, `field` или `param`):

```cs
public record Person([property: JsonPropertyName("firstName")] string FirstName, 
    [property: JsonPropertyName("lastName")] string LastName);
```

Если сгенерированное автоматически определённое свойство это не то, что вам нужно, можно его переопределить, создав поле/свойство с тем же именем вручную. Например, можно изменить видимость или добавить реализацию методов `get`/`set`. Объявляя поле/свойство вручную, его обязательно нужно инициализировать позиционным параметром.

Рекорд не обязан содержать позиционных параметров, а может смешивать позиционные параметры и обычные поля/свойства.

Свойства, которые генерирует компилятор из позиционных параметров - публичны. [[Модификаторы доступа]] можно редактировать на свойствах, объявленных явно.

# иммутабельность

*позиционный рекорд класс* и *позиционная рекорд ридонли структура* генерируют init-only свойства. *позиционная рекорд структура* генерирует поля, доступные для перезаписи.

Init-only свойства дают поверхностную иммутабельность. После инициализации значение значимого типа или ссылка на объект ссылочного типа не могут быть изменены, но данные, на которые ссылается поле ссылочного типа могут быть изменены.

# равенство

по умолчанию (если не переопределять методы равенства), равенство двух объектов определяется следующим образом:

- Для `class` два объекта равны, если они обращаются к одному объекту в памяти
- Для `struct` два объекта равны, если они одного типа и хранят одинаковые значения
- Для типов с модификатором `record` два объекта равны, если они одного типа и хранят одинаковые значения

Если в рекорд вложен ссылочный тип, он будет сравниваться по ссылке. В случае с коллекциями будет сгенерировано предупреждение о "suspicious equality", говорящее о том, что будет использоваться сравнение по ссылке, вместо (возможно ожидаемого) сравнения элементов. Для того, чтобы коллекции сравнивались по их элементам понадобится вручную переопределить методы `PrintMembers`, `GetHashCode` и `Equals`.

Определение равенства у `record struct` и `struct` одинаковое, за тем исключением, что для структуры используется `ValueType.Equals(Object)`, которое полагается на рефлексию, а для `record struct` компилятор генерирует код, полагающийся на объявленные члены.

В некоторых контекстах требуется проверка на равенство по ссылке. Так, EF Core полагается на сравнение по ссылке для определения существования одного объекта для каждой сущности. По этой причине рекорды и рекорд структуры не подходят для описания моделей в EF Core.

```cs
public record Person(string FirstName, string LastName, string[] PhoneNumbers);

public static void Main()
{
    var phoneNumbers = new string[2];
    Person person1 = new("Nancy", "Davolio", phoneNumbers);
    Person person2 = new("Nancy", "Davolio", phoneNumbers);
    Console.WriteLine(person1 == person2); // output: True

    person1.PhoneNumbers[0] = "555-1234";
    Console.WriteLine(person1 == person2); // output: True

    Console.WriteLine(ReferenceEquals(person1, person2)); // output: False
}
```

Чтобы реализовать сравнение по значению компилятор генерирует несколько методов, включая:

- Перегрузку `Object.Equals(Object)`. Будет считаться ошибкой ручная перегрузка этого метода. Этот метод используется в качестве основы для статического метода `Object.Equals(Object, Object)`, когда оба параметра не равны null.
- `virtual` или `sealed` `Equals(R? other)`, где `R` - тип рекорда. Этот метод реализует `IEquatable<T>`. Этот метод может быть объявлен явно.
- Если тип рекорда наследован от `Base`, генерируется `Equals(Base? other)`. Этот метод не может быть объявлен явно. Если предоставляется своя реализация `Equals(R? other)`, нужно так же переопределить `GetHashCode`.
- Перегрузка `Object.GetHashCode()`. Этот метод может быть объявлен явно.
- Перегрузка операторов `==` и `!=`. Эти операторы не могут быть объявлены явно.

# неразрушающие модификации

Можно использовать выражение `with`, чтобы создать новый объект с частичными изменениями полей.

# наследование

применимо только к `record class`. Рекорд может быть наследован от другого рекорда, но не от класса. Так же класс не может наследоваться от рекорда.

## позиционные параметры в унаследованных типах

унаследованный тип объявляет все позиционные параметры из первичного конструктора базового рекорда. унаследованный тип не скрывает исходные, а просто генерирует свойства для параметров, которых не было в базовом рекорде.

```cs
public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
public static void Main()
{
    Person teacher = new Teacher("Nancy", "Davolio", 3);
    Console.WriteLine(teacher);
    // output: Teacher { FirstName = Nancy, LastName = Davolio, Grade = 3 }
}
```

## равенство в цепочках наследования

чтобы две переменные были равны, нужно, чтобы в рантайме они были одного типа. Типы переменных, в которых находятся эти значения могут быть разными:

```cs
public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
public record Student(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
public static void Main()
{
    Person teacher = new Teacher("Nancy", "Davolio", 3);
    Person student = new Student("Nancy", "Davolio", 3);
    Console.WriteLine(teacher == student); // output: False

    Student student2 = new Student("Nancy", "Davolio", 3);
    Console.WriteLine(student2 == student); // output: True
}
```

Чтобы обеспечить это поведение компилятор генерирует свойство `EqualityContract`, которое возвращает тип текущего рекорда. Оно позволяет операторам равенства сравнивать рантайм типы объектов.

Когда сравниваются два объекта унаследованного типа, сгенерированные операторы равенства сравнивают все члены базового и наследованного типов. Сгенерированный `GetHashCode` метод использует `GetHashCode` каждого из членов базового и наследованного типов. Понятие "члены" включает как объявленные поля, так и сгенерированные внутренние поля для любых автоматически реализованных свойств.

## выражение `with` в унаследованных типах

результат выражения with того же рантайм типа, что и операнд выражения. все поля рантайм типа копируются, но изменить значения можно только типа времени компиляции:

```cs
public record Point(int X, int Y)
{
    public int Zbase { get; set; }
};
public record NamedPoint(string Name, int X, int Y) : Point(X, Y)
{
    public int Zderived { get; set; }
};

public static void Main()
{
    Point p1 = new NamedPoint("A", 1, 2) { Zbase = 3, Zderived = 4 };

    Point p2 = p1 with { X = 5, Y = 6, Zbase = 7 }; // Can't set Name or Zderived
    Console.WriteLine(p2 is NamedPoint);  // output: True
    Console.WriteLine(p2);
    // output: NamedPoint { X = 5, Y = 6, Zbase = 7, Name = A, Zderived = 4 }

    Point p3 = (NamedPoint)p1 with { Name = "B", X = 5, Y = 6, Zbase = 7, Zderived = 8 };
    Console.WriteLine(p3);
    // output: NamedPoint { X = 5, Y = 6, Zbase = 7, Name = B, Zderived = 8 }
}
```

Вот краткий пересказ предоставленной документации:

## Форматирование `PrintMembers` в унаследованных рекордах

Метод `PrintMembers`, автоматически генерируемый для унаследованных типов рекордов, вызывает свою базовую реализацию. Это означает, что вывод `ToString()` включает все публичные свойства и поля как производного, так и базового типа.

Пример:
```csharp
public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
public record Student(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);

public static void Main()
{
    Person teacher = new Teacher("Nancy", "Davolio", 3);
    Console.WriteLine(teacher);
    // output: Teacher { FirstName = Nancy, LastName = Davolio, Grade = 3 }
}
```

Вы можете предоставить собственную реализацию метода `PrintMembers`. В этом случае используйте одну из следующих сигнатур:
*   Для `sealed` рекорда, который наследуется от `object` (не объявляет базовый рекорд): `private bool PrintMembers(StringBuilder builder);`
*   Для `sealed` рекорда, который наследуется от другого рекорда: `protected override bool PrintMembers(StringBuilder builder);`
*   Для не `sealed` рекорда, который наследуется от `object`: `protected virtual bool PrintMembers(StringBuilder builder);`
*   Для не `sealed` рекорда, который наследуется от другого рекорда: `protected override bool PrintMembers(StringBuilder builder);`

Пример пользовательской реализации `PrintMembers`:
```csharp
public abstract record Person(string FirstName, string LastName, string[] PhoneNumbers)
{
    protected virtual bool PrintMembers(StringBuilder stringBuilder)
    {
        stringBuilder.Append($"FirstName = {FirstName}, LastName = {LastName}, ");
        stringBuilder.Append($"PhoneNumber1 = {PhoneNumbers[0]}, PhoneNumber2 = {PhoneNumbers[1]}");
        return true;
    }
}

public record Teacher(string FirstName, string LastName, string[] PhoneNumbers, int Grade)
    : Person(FirstName, LastName, PhoneNumbers)
{
    protected override bool PrintMembers(StringBuilder stringBuilder)
    {
        if (base.PrintMembers(stringBuilder))
        {
            stringBuilder.Append(", ");
        };
        stringBuilder.Append($"Grade = {Grade}");
        return true;
    }
};

public static void Main()
{
    Person teacher = new Teacher("Nancy", "Davolio", new string[2] { "555-1234", "555-6789" }, 3);
    Console.WriteLine(teacher);
    // output: Teacher { FirstName = Nancy, LastName = Davolio, PhoneNumber1 = 555-1234, PhoneNumber2 = 555-6789, Grade = 3 }
}
```

**Примечание:** Компилятор генерирует `PrintMembers` в производных записях, даже если базовый тип `ToString` запечатан. Вы также можете создать собственную реализацию `PrintMembers`.

## Поведение деконструктора в производных записях

Метод `Deconstruct` производной записи возвращает значения всех позиционных свойств типа, определенного на этапе компиляции. Если тип переменной является базовой записью, деконструируются только свойства базовой записи, если объект не приведен к производному типу.

Пример:
```csharp
public abstract record Person(string FirstName, string LastName);
public record Teacher(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);
public record Student(string FirstName, string LastName, int Grade)
    : Person(FirstName, LastName);

public static void Main()
{
    Person teacher = new Teacher("Nancy", "Davolio", 3);
    var (firstName, lastName) = teacher; // Doesn't deconstruct Grade
    Console.WriteLine($"{firstName}, {lastName}");// output: Nancy, Davolio

    var (fName, lName, grade) = (Teacher)teacher;
    Console.WriteLine($"{fName}, {lName}, {grade}");// output: Nancy, Davolio, 3
}
```

# Универсальные ограничения (Generic constraints)

Ключевое слово `record` является модификатором для типа `class` или `struct`. Добавление модификатора `record` включает описанное выше поведение. Не существует универсального ограничения, требующего, чтобы тип был записью. `record class` удовлетворяет ограничению `class`. `record struct` удовлетворяет ограничению `struct`.