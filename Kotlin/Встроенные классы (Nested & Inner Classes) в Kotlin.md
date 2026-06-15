
## 1. Основные понятия

## Что такое встроенный класс?

- Класс, объявленный **внутри другого класса**
    
- В Kotlin называется **nested class** (по умолчанию)
    
- Может быть **inner class** (если объявлен с ключевым словом `inner`)
    

## Типы встроенных классов:

|Тип|Синтаксис|Доступ к внешнему классу|
|---|---|---|
|**Nested class**|`class Outer { class Inner { } }`|❌ Нет доступа к членам внешнего|
|**Inner class**|`class Outer { inner class Inner { } }`|✅ Есть доступ к членам внешнего|

---

## 2. Nested class (встроенный класс по умолчанию)

## Синтаксис:

kotlin

`class Outer {     private val outerField = "outer"         class Nested {        fun access(): String {            // ❌ Нет доступа к outerField:            // return outerField  // Error                         return "nested only"        }    } }`

## Особенности:

- **По умолчанию** любой класс внутри другого — `nested`
    
- Не имеет доступа к членам внешнего класса
    
- Аналог `static inner class` в Java
    
- Может содержать только `static` члены (в терминологии Java)
    

## Использование:

kotlin

`val nested = Outer.Nested()  // Создание без экземпляра внешнего`

---

## 3. Inner class (внутренний класс с доступом)

## Синтаксис:

kotlin

`class Outer {     private val outerField = "outer"         fun outerMethod() = "method"         inner class Inner {        fun access(): String {            // ✅ Доступ к членам внешнего:            return outerField + outerMethod()        }    } }`

## Особенности:

- Нужно ключевое слово **`inner`**
    
- Имеет доступ к **всем** членам внешнего класса ( включая `private`)
    
- Аналог `inner class` в Java (не static)
    
- Содержит неявную ссылку на внешний класс
    

## Создание экземпляра:

kotlin

`val outer = Outer() val inner = outer.Inner()  // ❗ Требуется экземпляр внешнего класса`

---

## 4. Сравнение: Nested vs Inner

|Характеристика|Nested class|Inner class|
|---|---|---|
|**Ключевое слово**|Не нужно|`inner`|
|**Доступ к внешнему**|❌ Нет|✅ Да|
|**Требуется экземпляр внешнего**|❌ Нет|✅ Да|
|**Аналог в Java**|`static inner class`|`inner class`|
|**Может быть abstract**|✅ Да|✅ Да|
|**Может быть open**|✅ Да|✅ Да|

---

## 5. Доступ к членам внешнего класса

## Вложенный `inner` класс:

kotlin

`class Outer {     private val privateField = "private"    val publicField = "public"         inner class Inner {        fun show(): String {            return privateField + publicField  // ✅ Всё доступно        }    } }`

## Явный доступ через `this@Outer`:

kotlin

`class Outer {     private val value = "outer"         inner class Inner {        private val value = "inner"                 fun show(): String {            val innerValue = value           // ✅ value из Inner            val outerValue = this@Outer.value  // ✅ value из Outer                         return innerValue + outerValue        }    } }`

**`this@Outer`** — явная ссылка на внешний класс.

---

## 6. Вложенные классы с несколькими уровнями

kotlin

`class A {     val aField = "A"         inner class B {        val bField = "B"                 inner class C {            val cField = "C"                         fun show(): String {                // ✅ Доступ к всем уровням:                return aField + bField + cField            }        }    } } val a = A() val b = a.B() val c = b.C() c.show()  // "ABC"`

---

## 7. Встроенные классы + `companion object`

## `companion object` может создавать `inner` классы:

kotlin

`class Outer {     private val secret = "hidden"         companion object {        fun createInner(): Outer.Inner {            // ❌ Не работает:            // return Outer().Inner()  // Нет экземпляра                         // ✅ Нужно:            val outer = Outer()            return outer.Inner()        }    }         inner class Inner {        fun getSecret() = secret  // ✅ Доступ к private    } }`

## Но `companion object` сам не `inner`:

kotlin

`class Outer {     private val secret = "hidden"         companion object {        fun access(outer: Outer): String {            // ✅ Доступ к private внешнего:            return outer.secret        }    } }`

---

## 8. Абстрактные и open встроенные классы

kotlin

`abstract class Outer {     abstract fun outerMethod(): String         // ✅ Nested class может быть abstract:    class Nested {        abstract fun nestedMethod(): String    }         // ✅ Inner class может быть abstract:    inner class Inner {        abstract fun innerMethod(): String    } }`

---

## 9. Когда использовать

## ✅ Используйте `nested class` (без `inner`):

- Класс логически связан с внешним, но **не зависит** от его состояния
    
- Фабричные методы, константы, helpers
    
- Аналог `static class` в Java
    

kotlin

`class User {     val name: String         class Builder {        var name: String = ""                 fun build() = User(name)    } }`

## ✅ Используйте `inner class`:

- Класс **нуждаеться** в доступе к членам внешнего класса
    
- Event listeners, callback'ы
    
- Классы, зависящие от состояния внешнего
    

kotlin

`class NodeList {     private val items = mutableListOf<String>()         inner class Node {        val index: Int                 fun getItem() = items[index]  // ✅ Доступ к items внешнего    } }`

---

## 10. Когда НЕ использовать

## ❌ Не используйте встроенные классы, если:

- Класс не связан логически с внешним
    
- Класс должен быть публичным и независимым
    
- Лучше сделать **отдельный топ-левел класс**
    

kotlin

`// ❌ Плохо: class Outer {     class DatabaseConnection {  // Не связан с Outer        // ...    } } // ✅ Лучше: class DatabaseConnection {  // Отдельный класс     // ... }`

---

## 11. Особенности и ограничения

## Ограничения `inner` класса:

kotlin

`class Outer {     inner class Inner {        // ❌ НЕ может быть:        // top-level property        // global function                 // ✅ Но может:        // nested class        // property        // method    } }`

## `inner` class и наследование:

kotlin

`open class Outer {     inner class Inner {        open fun method() = "inner"    } } class DerivedOuter : Outer() {     inner class DerivedInner : Inner() {        override fun method() = "derived"    } }`

---

## 12. Сравнение с Java

|Kotlin|Java|Доступ к `private`|
|---|---|---|
|`class Outer { class Nested { } }`|`class Outer { static class Inner { } }`|❌ Нет|
|`class Outer { inner class Inner { } }`|`class Outer { class Inner { } }`|✅ Да|

**Важно:** В Java `static` inner class = Kotlin `nested`, а обычный `inner` = Kotlin с `inner`.

---

## 13. Практические паттерны

## Паттерн 1: Builder (nested class)

kotlin

`class Request {     val url: String    val method: String         class Builder {        private var url: String = ""        private var method: String = "GET"                 fun setUrl(u: String) = apply { url = u }        fun setMethod(m: String) = apply { method = m }                 fun build() = Request(url, method)    } } val request = Request.Builder()     .setUrl("https://api.com")    .setMethod("POST")    .build()`

## Паттерн 2: Iterator (inner class)

kotlin

`class Collection(val items: List<String>) {     inner class Iterator {        private var index = 0                 fun hasNext() = index < items.size        fun next() = items[index++]    }         fun createIterator() = Iterator() }`

## Паттерн 3: Callback (inner class)

kotlin

`class ApiService {     private var callback: Callback? = null         interface Callback {        fun onSuccess(data: String)        fun onError(e: Exception)    }         inner class DefaultCallback : Callback {        override fun onSuccess(data: String) {            // ✅ Доступ к members ApiService            log("Success: $data")        }                 override fun onError(e: Exception) {            log("Error: ${e.message}")        }    }         private fun log(msg: String) = println(msg) }`

---

## 14. Краткая сводка

|Ситуация|Что использовать|
|---|---|
|Класс не зависит от внешнего|`nested class` (без `inner`)|
|Класс нужен для access к внешнему|`inner class` (с `inner`)|
|Статические helpers|`companion object`|
|Независимый публичный класс|Топ-левел класс|

---

## 15. Частые ошибки

## Ошибка 1: Создание `inner` без внешнего

kotlin

`class Outer {     inner class Inner { } } // ❌ Не работает: val inner = Outer.Inner()  // Error // ✅ Правильно: val outer = Outer() val inner = outer.Inner()`

## Ошибка 2: Попытка доступа в `nested`

kotlin

`class Outer {     private val secret = "hidden"         class Nested {        // ❌ Не работает:        // fun get() = secret    } }`

## Ошибка 3: Забываешь `inner`

kotlin

`class Outer {     val value = "outer"         class Inner {  // ❌ Nested, не inner        fun get() = value  // Error: value not accessible    }         inner class CorrectInner {  // ✅ Правильно        fun get() = value  // Works    } }`

---

**Где найти в документации:**  
[Kotlin — Nested classes](https://kotlinlang.org/docs/classes.html#nested-classes)