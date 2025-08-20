---
stage: 1
status: done
---
Тип `dynamic` статический, но объект типа `dynamic` пропускает статическую типизацию. В большинстве случаев он ведёт себя как будто имеет тип `object`. Компилятор предполагает, что `dynamic` поддерживает любую операцию. Так, не нужно понимать, откуда взялось значение - из COM API, из динамического языка, как например IronPython, из HTML, из рефлексии или откуда бы то ни было ещё. Впрочем, если код некорректный, ошибки посыпятся в рантайме, компилятор о них знать не будет.

Компилятор собирает информацию об операции и использует её затем во время выполнения. В том числе переменные типа `dynamic` переводятся в тип `object`. Так, тип `dynamic` существует только во время компиляции, но в не рантайме.

Результат большинства операций с `dynamic` тоже `dynamic`. Операции, в результате которых получится не `dynamic` включают:

- Переводы из `dynamic` в другие типы
- Вызовы конструктора, которые включают аргументы типа `dynamic`

Любой тип можно неявно перевести в `dynamic`:

```cs
dynamic d1 = 7;
dynamic d2 = "a string";
dynamic d3 = System.DateTime.Today;
dynamic d4 = System.Diagnostics.Process.GetProcesses();
```

Так же можно динамически переводить `dynamic` в любой тип:

```cs
int i = d1;
string str = d2;
DateTime dt = d3;
System.Diagnostics.Process[] procs = d4;
```

Разрешение перегрузок так же происходит в рантайме, если один или больше аргументов метода типа `dynamic`, или владелец вызываемого метода сам типа `dynamic`. В этих случаях не происходит ошибок компиляции, вместо этого выбрасывается исключение времени выполнения:

```cs
static void Main(string[] args)
{
    ExampleClass ec = new ExampleClass();
    // The following call to exampleMethod1 causes a compiler error
    // if exampleMethod1 has only one parameter. Uncomment the line
    // to see the error.
    //ec.exampleMethod1(10, 4);

    dynamic dynamic_ec = new ExampleClass();
    // The following line is not identified as an error by the
    // compiler, but it causes a run-time exception.
    dynamic_ec.exampleMethod1(10, 4);

    // The following calls also do not cause compiler errors, whether
    // appropriate methods exist or not.
    dynamic_ec.someMethod("some argument", 7, null);
    dynamic_ec.nonexistentMethod();

	// Valid.
	ec.exampleMethod2("a string");
	
	// The following statement does not cause a compiler error, even though ec is not
	// dynamic. A run-time exception is raised because the run-time type of d1 is int.
	ec.exampleMethod2(d1);
	// The following statement does cause a compiler error.
	//ec.exampleMethod2(7);
}

class ExampleClass
{
    public ExampleClass() { }
    public ExampleClass(int v) { }

    public void exampleMethod1(int i) { }

    public void exampleMethod2(string str) { }
}
```