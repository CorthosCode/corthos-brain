
## 1. Что такое `value class`?

## Определение:

- **`value class`** (value-класс) — класс с модификатором `value` и аннотацией `@JvmInline`
    
- Компилятор **разворачивает** класс в байт-коде, заменяя его на **вложенное поле**
    
- Сочетает **производительность примитивов** + **type-safety полноценных классов**
    

## Основная идея:

kotlin

`// Обычный класс — создаётся объект class Age(val value: Int)  // new Age(25) // Value класс — объект НЕ создаётся, только Int @JvmInline value class Age(val value: Int)  // просто 25`

---

## 2. Синтаксис

## Минимальный пример:

kotlin

`@JvmInline value class Age(val value: Int)`

## Полноценный пример:

kotlin

`@JvmInline value class Email(val value: String) {     init {        require(value.contains("@")) { "Email must contain @" }    }         fun isValid(): Boolean = value.contains("@") }`

---

## 3. Ключевые особенности

## ✅ Что **МОЖЕТ** делать `value class`:

|Возможность|Пример|
|---|---|
|**Одно свойство**|`val value: Int`|
|**Методы**|`fun isValid() = ...`|
|**Реализация интерфейсов**|`: Comparable<Age>`|
|**`init` блок**|`init { require(...) }`|
|**Extension properties/methods**|`fun Age.isAdult() = ...`|

## ❌ Что **НЕ МОЖЕТ** делать `value class`:

|Ограничение|Пример|
|---|---|
|**Наследование**|`class A : B` — нельзя|
|**Быть наследуемым**|`open value class` — нельзя|
|**Несколько свойств**|`val a: Int, val b: String` — нельзя|
|**Nullable свойство**|`val value: Int?` — нельзя|
|**`lateinit`**|`lateinit val value` — нельзя|
|**`abstract`**|`abstract value class` — нельзя|
|**`inner`**|`inner value class` — нельзя|

---

## 4. Ограничения подробно

## Ограничение 1: Только одно свойство

kotlin

`// ✅ Правильно: @JvmInline value class Age(val value: Int) // ❌ Не работает: @JvmInline value class Person(val name: String, val age: Int)  // Error: more than one property`

## Ограничение 2: Свойство не может быть nullable

kotlin

`// ✅ Правильно: @JvmInline value class Age(val value: Int) // ❌ Не работает: @JvmInline value class Age(val value: Int?)  // Error: nullable type`

## Ограничение 3: Наследование запрещено

kotlin

`// ❌ Не работает: open class Base @JvmInline value class Derived(val value: Int) : Base()  // Error: inheritance // ❌ Не работает: @JvmInline open value class Base(val value: Int)  // Error: cannot be open`

## Ограничение 4: Не может быть `inner`

kotlin

`class Outer {     // ❌ Не работает:    @JvmInline    inner value class Inner(val value: Int)  // Error: inner }`

---

## 5. Производительность

## Сравнение в байт-коде:

## Обычный класс:

kotlin

`class Age(val value: Int) val age = Age(25)  // new Age(25) — объект в памяти`

**В байт-коде:** `new Age, invoke constructor`

## Value класс:

kotlin

`@JvmInline value class Age(val value: Int) val age = Age(25)  // просто 25 — нет объекта`

**В байт-коде:** `int 25` (примитив)

## Когда создаётся объект?

|Ситуация|Value class|
|---|---|
|**Локальная переменная**|❌ Нет (примитив)|
|**Параметр метода**|❌ Нет (примитив)|
|**Поле класса**|✅ Да (обёртка)|
|**Рефлексия**|✅ Да (обёртка)|
|**Generic тип**|✅ Да (обёртка)|

kotlin

`// ❌ Создаётся объект: class Container {     val age: Age = Age(25)  // Age обёрнут в объект } // ❌ Создаётся объект: fun process(age: Age) { ... }  // При вызове из Java // ✅ Не создаётся: val age = Age(25)  // Просто Int process(age)       // Просто Int`

---

## 6. Type Safety

## Проблема без `value class`:

kotlin

`// Можно передать всё что угодно: fun setAge(age: Int) { ... } setAge(25)     // ✅ OK setAge(-5)     // ❌ Но компилятор не знает! setAge(1000)   // ❌ Ошибка логики`

## Решение с `value class`:

kotlin

`@JvmInline value class Age(val value: Int) {     init {        require(value >= 0) { "age must be >= 0" }        require(value <= 150) { "age must be <= 150" }    } } fun setAge(age: Age) { ... } setAge(Age(25))   // ✅ OK setAge(Age(-5))   // ❌ IllegalArgumentException при создании`

**Преимущества:**

- Нельзя передать `Int` вместо `Age`
    
- Валидация в `init`
    
- Чёткая семантика в типах
    

---

## 7. Реализация интерфейсов

kotlin

`@JvmInline value class Age(val value: Int) : Comparable<Age> {     override fun compareTo(other: Age): Int {        return value.compareTo(other.value)    } } val a1 = Age(25) val a2 = Age(30)  println(a1.compareTo(a2))  // -1 println(a1 < a2)           // true (использует compareTo)`

## Пример с `Comparable`:

kotlin

`@JvmInline value class Money(val value: Int) : Comparable<Money> {     override fun compareTo(other: Money): Int = value.compareTo(other.value)         operator fun plus(other: Money): Money = Money(value + other.value)    operator fun minus(other: Money): Money = Money(value - other.value) }`

---

## 8. Практические примеры

## Пример 1: Validation в `init`

kotlin

`@JvmInline value class Email(val value: String) {     init {        require(value.contains("@")) { "Email must contain @" }        require(value.length > 5) { "Email too short" }    }         fun isValid(): Boolean = true  // Если создан — всегда valid } val email = Email("test@example.com")  // ✅ OK val invalid = Email("invalid")         // ❌ IllegalArgumentException`

## Пример 2: Domain тип

kotlin

`@JvmInline value class UserId(val value: String) @JvmInline value class OrderId(val value: String) // Нельзя заменить: fun getUser(id: UserId) { ... } fun getOrder(id: OrderId) { ... } getUser(UserId("123"))  // ✅ OK getUser(OrderId("123")) // ❌ Error: тип не совпадает`

## Пример 3: Units

kotlin

`@JvmInline value class Distance(val value: Double) {     init {        require(value >= 0) { "Distance must be >= 0" }    }         operator fun plus(other: Distance): Distance =        Distance(value + other.value)         operator fun times(factor: Double): Distance =        Distance(value * factor) } val d1 = Distance(10.0) val d2 = Distance(5.0) val total = d1 + d2 * 2  // 20.0`

## Пример 4: Temperature с валидацией

kotlin

`@JvmInline value class Temperature(val value: Int) {     init {        require(value >= -273) { "Cannot be below absolute zero" }    }         fun toCelsius(): Int = value    fun toKelvin(): Int = value + 273 }`

---

## 9. Value class в стандартной библиотеке

## Примеры из Kotlin stdlib:

kotlin

`// Kotlin стандартная библиотека использует value class: @JvmInline value class Char(val value: Char)  // Упрощённо @JvmInline value class Duration(val length: Long) @JvmInline value class FontWeight(val weight: Int)`

---

## 10. Интероперабельность с Java

## Вызов из Java:

kotlin

`// Kotlin: @JvmInline value class Age(val value: Int) // Java: Age age = new Age(25);  // Создаётся обёртка int value = age.getValue();  // Получение значения`

## Аннотация `@JvmInline`:

- **Обязательна** для JVM (Kotlin/JVM)
    
- Нужна, чтобы компилятор знал: разворачивать класс
    
- Для Kotlin Native/JS не всегда нужна
    

kotlin

`// Обязательно для JVM: @JvmInline value class Age(val value: Int) // Для Kotlin Native/JS: value class Age(val value: Int)  // Без @JvmInline`

---

## 11. Когда использовать

## ✅ Используйте `value class`, если:

|Ситуация|Почему|
|---|---|
|**Domain тип поверх примитива**|Type safety + валидация|
|**Минимальная обёртка**|Нет runtime overhead|
|**Частые операции**|Производительность как примитив|
|**Семантика типа**|`UserId` vs `String`|
|**Валидация при создании**|`init` блок гарантирует корректность|

## ❌ Не используйте `value class`, если:

|Ситуация|Почему|
|---|---|
|**Несколько свойств**|Только одно поле|
|**Нужно наследование**|Запрещено|
|**Nullable тип**|Нельзя `Int?`|
|**Generic параметры**|Создаётся обёртка|
|**Поле класса**|Создаётся обёртка|
|**Рефлексия**|Создаётся обёртка|

---

## 12. Когда НЕ использовать

## Не использовать для:

kotlin

`// ❌ Слишком сложно: @JvmInline value class Person(val name: String, val age: Int)  // Error // ❌ Nullable: @JvmInline value class OptionalAge(val value: Int?)  // Error // ❌ Наследование: open class Base @JvmInline value class Derived(val v: Int) : Base()  // Error // ✅ Лучше обычный класс: class Person(val name: String, val age: Int) class OptionalAge(val value: Int?)`

---

## 13. Сравнение с аналогами

|Характеристика|Обычный класс|`value class`|Примитив|
|---|---|---|---|
|**Создание объекта**|✅ Да|❌ Нет (локально)|❌ Нет|
|**Multiple properties**|✅ Да|❌ Нет|❌ Нет|
|**Наследование**|✅ Да|❌ Нет|❌ Нет|
|**Type safety**|✅ Да|✅ Да|❌ Нет|
|**Валидация**|✅ Да|✅ Да|❌ Нет|
|**Интерфейсы**|✅ Да|✅ Да|❌ Нет|
|**Runtime overhead**|Высокий|Минимальный|Нет|

---

## 14. Частые ошибки

## Ошибка 1: Несколько свойств

kotlin

`// ❌ Не работает: @JvmInline value class Point(val x: Int, val y: Int)  // Error // ✅ Лучше обычный класс: class Point(val x: Int, val y: Int)`

## Ошибка 2: Nullable тип

kotlin

`// ❌ Не работает: @JvmInline value class Age(val value: Int?)  // Error // ✅ Лучше: @JvmInline value class Age(val value: Int)  // Ненулевое значение`

## Ошибка 3: Забываешь `@JvmInline`

kotlin

`// ❌ Не работает на JVM: value class Age(val value: Int)  // Error: @JvmInline required // ✅ Правильно: @JvmInline value class Age(val value: Int)`

## Ошибка 4: Пытаешься наследовать

kotlin

`// ❌ Не работает: open class Base(val v: Int) @JvmInline value class Derived(val v: Int) : Base(v)  // Error // ✅ Лучше: @JvmInline value class Age(val value: Int)  // Без наследования`

---

## 15. Краткая сводка

|Что|Как|
|---|---|
|**Объявить**|`@JvmInline value class Name(val value: T)`|
|**Свойство**|Ровно одно, не nullable|
|**Методы**|Можно добавлять|
|**Interfaces**|Можно реализовывать|
|**Наследование**|❌ Нельзя|
|**init блок**|✅ Можно (валидация)|
|**Байт-код**|Примитив (локально)|

---

## 16. Преимущества и недостатки

## ✅ Преимущества:

- **Type safety** — нельзя заменить тип
    
- **Валидация** — `init` гарантирует корректность
    
- **Производительность** — как примитив
    
- **Семантика** — чёткая в коде (`UserId` vs `String`)
    
- **Методы** — можно добавлять логику
    

## ❌ Недостатки:

- **Одно свойство** — ограничение
    
- **Нельзя наследовать** — нет OOP
    
- **Nullable нельзя** — только `Int`, не `Int?`
    
- **Обёртка в некоторых случаях** — поле, generic, рефлексия
    

---

**Где найти в документации:**  
[Kotlin — Value classes](https://kotlinlang.org/docs/value-classes.html)