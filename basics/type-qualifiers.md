# Изменяемость

D является статически типизированным языком: начиная с момента объявления
переменной, её тип не может быть изменен. Это позволяет компилятору
предотвращать ошибки на раннем этапе и обеспечивать соблюдение ограничений во
время компиляции. Хорошая типобезопасность обеспечивает вам необходимую
поддержку, чтобы сделать большие программы более безопасными и более лёгкими в
обслуживании.

В языке D есть несколько квалификаторов типа, но наиболее используемыми являются
`const` и `immutable`.

### `immutable`

В дополнение к статической системе типов, D предоставляет квалификаторы типа
(иногда называемые также "конструкторы типа"), которые накладывают
дополнительные ограничения на определённые объекты. Например, `immutable` объект
может быть инициализирован только один раз, после чего любые изменения
запрещены.

    immutable int err = 5;
    // либо: immutable arr = 5; и int будет выведен.
    err = 5; // не скомпилируется

Таким образом, `immutable` объекты могут быть безопасно разделены между разными
потоками без использования синхронизации, так как они по определению никогда не
меняются. Это означает, что `immutable` объекты прекрасно кешируются.

### `const`

`const` объекты также не могут быть изменены. Это ограничение действительно
только для текущей области видимости. `const` указатель может быть создан как из
*mutable*, так и из `immutable` объекта. Это означает, что объект является
`const` для текщей области видимости, но кто-то ещё может изменить его из
отличающегося контекста. Общепринято, что API принимают `const` аргументы, чтобы
гарантировать неизменность входных данных и это позволяет одной и той же функции
обрабатывать как изменяемые, так и неизменяемые данные.

    void foo ( const char[] s )
    {
        // если не закомментрованно, то на следующей
        // строке будет ошибка (невозможно изменить костанту):
        // s[0] = 'x';

        import std.stdio;
        writeln(s);
    }

    // благодаря `const`, оба вызова будут корректны:
    foo("abcd"); // строка является immutable массивом
    foo("abcd".dup); // .dup возвращает изменяемую копию

И `immutable` и `const` _транзитивны_, обеспечивая, что если однажды `const`
было применено к типу, оно рекурсивно применяется к каждому субкомпоненту этого
типа.

### Подробнее

#### Основные ссылки

- [Immutable in _Programming in D_](http://ddili.org/ders/d.en/const_and_immutable.html)
- [Scopes in _Programming in D_](http://ddili.org/ders/d.en/name_space.html)

#### Дополнительные ссылки

- [const(FAQ)](https://dlang.org/const-faq.html)
- [Type qualifiers D](https://dlang.org/spec/const3.html)

## {SourceCode}

```d
import std.stdio;

void main()
{
    immutable forever = 100;
    // ОШИБКА:
    // forever = 5;
    writeln("forever: ",
        typeof(forever).stringof);

    const int* cForever = &forever;
    // ОШИБКА:
    // *cForever = 10;
    writeln("const* forever: ",
        typeof(cForever).stringof);

    int mutable = 100;
    writeln("mutable: ",
        typeof(mutable).stringof);
    mutable = 10; // Отлично
    const int* cMutable = &mutable; // Отлично
    // ОШИБКА:
    // *cMutable = 100;
    writeln("cMutable: ",
        typeof(cMutable).stringof);

    // ОШИБКА:
    // immutable int* imMutable = &mutable;
}
```