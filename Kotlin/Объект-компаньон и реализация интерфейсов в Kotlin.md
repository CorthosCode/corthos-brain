
## 1. Основные понятия

## Объект-компаньон (`companion object`)

-special объект, объявленный внутри класса

- Существует в единственном экземпляре (как static в Java)
    
- Доступен по имени класса: `ClassName.method()` или `ClassName.property`
    
- Может содержать методы, свойства и реализовывать интерфейсы
    

## Интерфейс в Kotlin

- Опребатывает набор методов, которые должны реализовать классы/объекты
    
- Не содержит состояния (только сигнатуры методов)
    
- Может быть реализован как классами, так и объектами
    

---

## 2. Ключевое утверждение

> **В объявлении объекта-компаньона можно реализовывать интерфейсы. Имя содержащего класса можно использовать непосредственно в качестве экземпляра объекта, реализующего интерфейс.**

**Что это значит:**

- `companion object` может объявлять `: InterfaceName`
    
- Имя класса автоматически ссылается на этот объект-компаньон
    
- Класс можно присвоить в переменную типа интерфейса без создания экземпляра
    

---

## 3. Примеры кода

## Базовый пример

kotlin

`interface Logger {     fun log(message: String) } class MyClass {     companion object : Logger {        override fun log(message: String) {            println("[LOG] $message")        }    } } // Использование val logger: Logger = MyClass  // MyClass — это companion object, не new MyClass() logger.log("Hello")           // Вывод: [LOG] Hello`

## Несколько интерфейсов

kotlin

`interface Reader {     fun read(): String } interface Writer {     fun write(data: String) } class FileHandler {     companion object : Reader, Writer {        override fun read(): String = "data"        override fun write(data: String) {            println("Writing: $data")        }    } } val reader: Reader = FileHandler val writer: Writer = FileHandler`

## С методами внутри компаньона

kotlin

`interface Service {     fun start()    fun stop() } class Database {     companion object : Service {        private var connected = false                 override fun start() {            connected = true            println("Database connected")        }                 override fun stop() {            connected = false            println("Database disconnected")        }                 // Дополнительный метод        fun isConnected(): Boolean = connected    } } // Использование Service db = Database db.start() println(Database.isConnected())  // true`

---

## 4. Механизм работы

## Как Kotlin обрабатывает это:

1. **Компиляция**: `companion object` компилируется как внутренний класс с именем `ClassName$companion`
    
2. **Обращение**: `ClassName` автоматически преобразуется в `ClassName.Companion`
    
3. **Типизация**: Если `companion object` реализует интерфейс, имя класса имеет этот тип
    

## Почему имя класса работает как экземпляр:

kotlin

`class MyClass {     companion object : Logger { ... } } // Эти выражения эквивалентны: val a: Logger = MyClass val b: Logger = MyClass.Companion // a и b — один и тот же объект`

---

## 5. Преимущества использования

## ✅ Статическое поведение через интерфейс

kotlin

`// Без companion object : interface class Utils {     fun log(msg: String) = println(msg) } val utils = Utils()  // Нужно создавать экземпляр utils.log("test") // С companion object : interface class Utils {     companion object : Logger {        override fun log(msg: String) = println(msg)    } } val logger: Logger = Utils  // Без создания экземпляра logger.log("test")`

## ✅ Фабричные методы через интерфейс

kotlin

`interface Factory<T> {     fun create(): T } class User {     companion object : Factory<User> {        override fun create(): User = User()    } } val factory: Factory<User> = User val user = factory.create()`

## ✅ Single Responsibility для статических операций

- Статическое поведение выделено в интерфейс
    
- Чёткая семантика через типизацию
    
- Можно тестировать как отдельный компонент
    

---

## 6. Ограничения и нюансы

## ❌ Не работает с параметрами класса

kotlin

`class MyClass(private val value: String) {     companion object : Logger { ... } } // MyClass нельзя использовать как Logger, если нужен экземпляр с параметрами`

## ❌ Приоритет свойств класса

Если у класса есть свойство с тем же именем, что и доступ к компаньону:

kotlin

`class MyClass {     val Companion: String = "instance"  // Свойство класса         companion object : Logger { ... } } // MyClass.Companion теперь ссылается на String, а не на companion object`

## ✅ Проверка типа

kotlin

`val logger: Logger = MyClass println(logger is MyClass.Companion)  // true`

---

## 7. Сравнение с другими языками

|Язык|Статический метод через интерфейс|Аналог в Kotlin|
|---|---|---|
|**Java**|❌ Не возможно (static не реализует интерфейсы)|`companion object : Interface`|
|**C#**|❌ Не возможно (static классы не реализуют интерфейсы)|`companion object : Interface`|
|**Kotlin**|✅ Возможно|`companion object : Interface`|

---

## 8. Практические паттерны использования

## Паттерн 1: Статический логгер

kotlin

`interface Logger {     fun log(msg: String) } class MyClass {     companion object : Logger {        override fun log(msg: String) {            println("${MyClass::class.simpleName}: $msg")        }    } } // Использование Logger.log("test")  // Через интерфейс MyClass.log("test") // Через имя класса`

## Паттерн 2: Фабрика

kotlin

`interface Creator<T> {     fun create(): T } class Config {     companion object : Creator<Config> {        override fun create(): Config = Config()                 fun withValue(v: String): Config = Config(v)    } } val creator: Creator<Config> = Config val config = creator.create()`

## Паттерн 3: Singleптон-подобное поведение

kotlin

`interface ResourceManager {     fun getResource(id: String): Any } class ResourceManagerImpl {     companion object : ResourceManager {        private val cache = mutableMapOf<String, Any>()                 override fun getResource(id: String): Any {            return cache.getOrElse(id) {                val resource = Any()                cache[id] = resource                resource            }        }    } }`

---

## 9. Частые ошибки

## Ошибка 1: Попытка создать экземпляр

kotlin

`// ❌ НЕПРАВИЛЬНО val logger: Logger = MyClass()  // MyClass() — это экземпляр, не companion // ✅ ПРАВИЛЬНО val logger: Logger = MyClass  // MyClass — это companion object`

## Ошибка 2: Константа вместо объекта

kotlin

`class MyClass {     companion object : Logger { ... } } // ❌ Не работает как ожидаемое val instance = MyClass()   val logger = instance  // Это экземпляр, не companion // ✅ Правильно val logger = MyClass`

---

## 10. Краткая сводка

|Что|Как|
|---|---|
|**Объявить**|`companion object : InterfaceName { ... }`|
|**Использовать**|`val varName: InterfaceName = ClassName`|
|**Доступ к методам**|`ClassName.method()` или `varName.method()`|
|**Тип объекта**|`ClassName` имеет тип `InterfaceName`|
|**Единственный экземпляр**|Да, как у любого `object`|

---

## 11. Когда использовать

## ✅ Используйте, если:

- Нужен статический метод, доступный через интерфейс
    
- Хотите избежать создания экземпляров для «статических» операций
    
- Нужен фабрика/логгер/менеджер с поведением интерфейса
    
- Хотите чёткую типизацию статического поведения
    

## ❌ Не используйте, если:

- Нужно состояние, зависящее от экземпляра класса
    
- Методы компаньона редко используются
    
- Есть проще альтернатива (например, top-level функции)
    

---

**Где найти в документации:**  
[Kotlin — Companion objects](https://kotlinlang.org/docs/object-declarations.html#companion-objects)