
## 1. Что такое sealed classes

**Sealed classes (и интерфейсы)** — это механизм ограничения наследования, появившийся в **Java 15 (preview), стабилен с Java 17**.

Они позволяют автору класса явно указать **какие классы могут наследовать (extends) или реализовывать (implements) данный класс/интерфейс**.

Это своего рода более гибкий и безопасный аналог `final` и `abstract`.

Пример:

```java
public sealed class Shape permits Circle, Rectangle, Square {}

public final class Circle extends Shape {}
public final class Rectangle extends Shape {}
public final class Square extends Shape {}
```

Здесь:

* `Shape` — sealed class.
* Только `Circle`, `Rectangle`, `Square` могут его расширять.
* Другие классы, даже находящиеся в том же пакете, **не могут**.


***

## 2. Модификаторы в иерархии sealed

У наследников sealed-класса есть строгое ограничение:

каждый из них **обязан** указать один из трёх модификаторов:

1. **`final`** — запрещает дальнейшее наследование.

```java
public final class Circle extends Shape {}
```

Всё, "ветка" на этом закончена.

1. **`sealed`** — разрешает только ограниченному списку наследников.

```java
public sealed class Polygon extends Shape permits Triangle, Pentagon {}
```

1. **`non-sealed`** — снимает ограничение и открывает иерархию дальше.

```java
public non-sealed class SpecialShape extends Shape {}
```

Таким образом, `sealed` создаёт **иерархию с чётким контрактом**, а `non-sealed` даёт "дырку" — из него можно наследовать кто угодно.


***

## 3. Правила sealed-классов

* `permits` можно указывать явно или компилятор сам подставит **все классы-наследники из того же пакета** (или модуля, если используется JPMS).
* Наследники должны находиться **в том же модуле**, а если модулей нет — **в том же пакете**.
* Если забыть модификатор (`final` / `sealed` / `non-sealed`), компилятор выдаст ошибку.
* sealed может быть и **классом**, и **интерфейсом**.


***

# 4. Sealed интерфейсы

Пример:

```java
public sealed interface Operation permits Addition, Subtraction {}

public final class Addition implements Operation {}
public final class Subtraction implements Operation {}
```

Здесь — то же самое, только через `implements`.


***

## 5. Зачем нужны sealed classes

✅ Гарантированная полнота switch/instanceof

Вместе с **pattern matching for switch** (Java 21) sealed-классы раскрываются очень мощно.

Компилятор **знает все возможные варианты** и может заставить программиста покрыть их все.

```java
public String describe(Shape shape) {
    return switch (shape) {
        case Circle c    -> "Circle";
        case Rectangle r -> "Rectangle";
        case Square s    -> "Square";
        // default не нужен — компилятор знает все варианты!
    };
}
```

Это даёт **экзистенциальную гарантию**, что ни один случай не будет пропущен.


***

## 6. Отличия от других механизмов

* `final` запрещает наследование полностью → sealed даёт частичный контроль.
* `enum` — это ограниченный набор значений, но:
  * у `enum` фиксированные экземпляры;
  * у sealed-классов фиксированные **типы**, но не фиксированные экземпляры.
* sealed + record часто используются вместе для выражения **алгебраических типов данных (ADT)**, как в функциональных языках.

Пример:

```java
public sealed interface Result<T> permits Success, Failure {}

public record Success<T>(T value) implements Result<T> {}
public record Failure<T>(String message) implements Result<T> {}
```

Это альтернатива `Optional`, `Either`, `Try` (из FP).


***

## 7. Продвинутые темы


### 7.1. Sealed + Records

Очень распространённая комбинация:

```java
public sealed interface Expr permits NumberExpr, AddExpr, MulExpr {}

public record NumberExpr(int value) implements Expr {}
public record AddExpr(Expr left, Expr right) implements Expr {}
public record MulExpr(Expr left, Expr right) implements Expr {}
```

→ можно строить дерево выражений и разбирать его безопасным switch-выражением.


### 7.2. Sealed + Pattern Matching

С Java 21 `switch` и `instanceof` работают особенно мощно:

```java
public int eval(Expr expr) {
    return switch (expr) {
        case NumberExpr n -> n.value();
        case AddExpr a    -> eval(a.left()) + eval(a.right());
        case MulExpr m    -> eval(m.left()) * eval(m.right());
    };
}
```

Компилятор знает, что default не нужен — все случаи покрыты.


### 7.3. Sealed + Generics

Sealed интерфейсы могут быть обобщёнными:

```java
public sealed interface Option<T> permits Some<T>, None<T> {}

public record Some<T>(T value) implements Option<T> {}
public record None<T>() implements Option<T> {}
```


### 7.4. Наследование через `non-sealed`

Иногда полезно "разрешить лазейку":

```java
public sealed class Animal permits Dog, Cat, Other {}

public final class Dog extends Animal {}
public final class Cat extends Animal {}
public non-sealed class Other extends Animal {}
```

Теперь кто угодно может расширить `Other`.


### 7.5. Модули и sealed

Если проект использует JPMS (Java Platform Module System):

* sealed-класс может быть наследован **только в том же модуле**.
* Это усиливает инкапсуляцию, позволяя ограничить API ещё жёстче.


### 7.6. Ограничение с reflection

С помощью reflection можно узнать:

```java
Class<?>[] permitted = Shape.class.getPermittedSubclasses();
```

→ так можно introspect sealed-иерархию во время выполнения.


### 7.7. Совместимость с Lombok, Jackson и др.

* Lombok поддерживает `sealed` начиная с 1.18.20.
* Jackson (и другие JSON-библиотеки) могут сериализовать sealed-иерархии, но иногда надо включать "type info".
* JPA и Hibernate пока не полностью поддерживают sealed-иерархии (нужны костыли).


***

## 8. Когда использовать sealed classes

* Когда набор подклассов **замкнутый и фиксированный**.
* Когда важно гарантировать исчерпывающие проверки (например, при pattern matching).
* Когда нужны **ADT-подобные конструкции** (Result, Option, Expr, State, Command).
* Когда проект большой, и нужно **контролировать расширение API**.

