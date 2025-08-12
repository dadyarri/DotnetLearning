---
tags:
  - этап1
  - todo
---
Тип `string` представляет собой последовательность из нуля или больше Unicode символов. `string` - алиас для `System.String` в .NET.

Хотя строка - это ссылочный тип, её операторы равенства переопределены, чтобы сравнивать значения объектов типа `string`, а не ссылки:

```cs
string a = "hello";
string b = "h";
// Append to contents of 'b'
b += "ello";
Console.WriteLine(a == b); // True
Console.WriteLine(object.ReferenceEquals(a, b)); // False
```

Оператор `+` конкатенирует строки.

Строки *неизменяемы* - содержимое объекта строкового типа не может быть изменено после создания объекта.