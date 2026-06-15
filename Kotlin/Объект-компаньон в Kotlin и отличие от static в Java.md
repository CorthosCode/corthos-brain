
## 1. Что такое объект-компаньон

- **`companion object`** — специальный объект, связанный с классом в Kotlin
    
- Принадлежит **классу**, а не его экземплярам
    
- Аналог `static` в Java, но с отличиями в доступе
    

kotlin

`class MyClass {     companion object {        fun staticMethod() = "статика"        val staticValue = 100    } }`

---

## 2. Ключевое правило доступа

## ✅ Как правильно:

kotlin

`MyClass.staticMethod()     // через имя класса MyClass.staticValue`

## ❌ Как НЕправильно:

kotlin

`val instance = MyClass() instance.staticMethod()    // ошибка компиляции! instance.staticValue       // ошибка компиляции!`

**Правило:** нельзя получать доступ к членам компаньона из экземпляра класса напрямую.

---

## 3. Отличие от Java static

|Характеристика|Java `static`|Kotlin `companion object`|
|---|---|---|
|Принадлежность|Классу|Классу|
|Доступ из экземпляра|✅ `instance.staticMethod()`|❌ только `Class.method()`|
|Доступ к приватным полям класса|❌ Нет|✅ **Да** (ключевое преимущество)|
|ООП-стиль|Безликие функции|Объект как член класса|

**Пример доступа к приватным:**

kotlin

`class MyClass(private val secret: String) {     companion object {        fun printSecret(obj: MyClass) {            println(obj.secret)  // ✅ работает!        }    } }`

---

## 4. Почему Kotlin отказался от static

1. **Более объектно-ориентированный подход** — поведение как член объекта
    
2. **Доступ к приватным членам** — companions могут читать `private` поля класса
    
3. **Можно реализовать интерфейс** — `companion object` может иметь типы
    
4. **Совместимость с Java** — при компиляции в `static`, но внутри Kotlin действуют свои правила
    

---

## 5. Практические примеры использования

## Фабричные методы

kotlin

`class User private constructor(val name: String) {     companion object {        fun create(name: String) = User(name)  // фабрика    } }`

## Константы класса

kotlin

`class HttpClient {     companion object {        const val DEFAULT_TIMEOUT = 3000        const val BASE_URL = "https://api.example.com"    } }`

## Валидаторы

kotlin

`class Email(val value: String) {     companion object {        fun isValid(value: String): Boolean {            return value.contains("@")        }    } }`

---

## 6. Частые ошибки

|Ошибка|Почему|Как исправить|
|---|---|---|
|`instance.companionMethod()`|Экземпляр не видит компаньон|`ClassName.companionMethod()`|
|Прямой вызов без имени класса|Отсутствует импорт|Использовать `ClassName.`|
|기대 컴파일 ошибка|Нарушено правило доступа|Перепроверить точку доступа|

---

## 7. Сводная таблица отличий

|Аспект|Kotlin companion|Java static|
|---|---|---|
|Объявление|`companion object { }`|`static { }` / `static field`|
|Доступ из instance|❌ Нет|✅ Да|
|Access to private|✅ Да|❌ Нет|
|Может реализовать интерфейс|✅ Да|❌ Нет|
|Композиция в Java|—|Компилируется в static|

---

## 8. Запомнить

> **Объект-компаньон принадлежит классу → доступ только через имя класса**

Это фундаментальное отличие от Java, где `static` можно вызывать из экземпляра.