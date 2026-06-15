
## 1. Что такое `internal`?

## Определение:

- **`internal`** — модификатор доступа в Kotlin
    
- Объявление с `internal` доступно **внутри того же модуля**
    
- **Недоступно** из других модулей
    

## Ключевая фраза:

> **Область видимости объявления типа `internal` ограничена модулем.**

---

## 2. Что такое «модуль»?

## Определение модуля в Kotlin:

|Контекст|Что считается модулем|
|---|---|
|**Kotlin/JVM**|Набор `.kt` файлов, скомпилированных вместе (например, один Gradle модуль)|
|**Android**|Один Gradle модуль (`:app`, `:core`, `:data`)|
|**Kotlin Build**|Один `sourceSet` в Gradle/Maven|
|**IDEA**|Один модуль проекта|

## Примеры модулей:

text

`// build.gradle — несколько модулей в проекте: project(':app')          // Модуль 1 project(':core')         // Модуль 2 project(':data')         // Модуль 3`

**Файлы внутри `:core`** — это **один модуль**.

---

## 3. Синтаксис

## Объявление с `internal`:

kotlin

`// internal класс internal class InternalClass {     internal fun internalMethod() = "internal"    internal val internalProperty = 42 } // internal функция internal fun internalFunction() = "internal" // internal свойство internal var internalVariable = "internal" // internal интерфейс internal interface InternalInterface {     fun doSomething() } // internal класс внутри другого класса class Outer {     internal inner class InternalInner { } }`

---

## 4. Сравнение модификаторов доступа

|Модификатор|Видимость|Аналог в Java|
|---|---|---|
|**`private`**|Только внутри того же класса/объекта|`private`|
|**`protected`**|Внутри класса + наследники|`protected`|
|**`internal`**|**Внутри модуля**|(нет аналога)|
|**`public`**|Из любого места|`public`|

## По умолчанию:

kotlin

`// По умолчанию = public: class MyClass {  // public class     fun myMethod() { }  // public method    val myProp = 42  // public property }`

---

## 5. Пример: `internal` в одном модуле

## Модуль `:core`:

kotlin

`// File: core/src/InternalHelper.kt internal class InternalHelper {     fun help() = "helping" } internal fun internalUtility() = "utility"`

kotlin

`// File: core/src/AnotherClass.kt class AnotherClass {     fun doSomething() {        // ✅ Доступ к internal из того же модуля:        val helper = InternalHelper()        helper.help()                 internalUtility()    } }`

**Вывод:** `internal` доступно **внутри модуля `:core`**.

---

## 6. Пример: `internal` из другого модуля не доступен

## Модуль `:app` (зависит от `:core`):

kotlin

`// File: app/src/AppClass.kt class AppClass {     fun doSomething() {        // ❌ НЕдоступно из другого модуля:        // val helper = InternalHelper()  // Error: unresolved        // internalUtility()  // Error: unresolved    } }`

**Вывод:** `internal` **НЕ видно** из модуля `:app`.

---

## 7. Практическое использование `internal`

## Сценарий 1: API библиотеки

kotlin

`// library/src/PublicAPI.kt public class PublicClient {     private val helper = InternalHelper()  // ✅ Внутри класса } // library/src/InternalHelper.kt internal class InternalHelper {     fun internalHelp() = "internal" } // library/src/AnotherInternal.kt internal class AnotherInternal {     fun useHelper() {        val h = InternalHelper()  // ✅ Внутри модуля library        h.internalHelp()    } }`

**Пользователь библиотеки** (другой модуль):

kotlin

`// user/src/UserCode.kt class UserCode {     fun use() {        val client = PublicClient()  // ✅ Public доступен                 // ❌ InternalHelper НЕ доступен:        // val h = InternalHelper()  // Error    } }`

---

## Сценарий 2: Разделение на public/internal API

kotlin

`// core/src/ApiClient.kt public class ApiClient {     public fun fetch(url: String) = "fetching"         internal fun internalFetch(url: String) = "internal fetch" } // core/src/InternalFetcher.kt internal class InternalFetcher {     fun fetch(client: ApiClient) {        // ✅ Доступ к internal из того же модуля:        client.internalFetch("/api")    } }`

kotlin

`// different-module/src/ExternalUser.kt class ExternalUser {     fun use() {        val client = ApiClient()        client.fetch("/api")  // ✅ Public                 // ❌ internal НЕ доступен:        // client.internalFetch("/api")  // Error    } }`

---

## Сценарий 3: Внутренняя логика модуля

kotlin

`// data/src/DataModule.kt internal class DataRepository {     fun getData() = "data" } internal class DataCache {     fun cache(data: String) = "cached" } // data/src/PublicDataProvider.kt public class PublicDataProvider {     private val repository = DataRepository()  // ✅ Внутри модуля    private val cache = DataCache()  // ✅ Внутри модуля         public fun provide(): String {        val data = repository.getData()        cache.cache(data)        return data    } }`

**Извне модуля:**

kotlin

`// app/src/App.kt class App {     fun start() {        val provider = PublicDataProvider()        provider.provide()  // ✅ Public                 // ❌ DataRepository и DataCache НЕ доступны:        // val repo = DataRepository()  // Error    } }`

---

## 8. `internal` для членов класса

kotlin

`public class PublicClass {     public fun publicMethod() = "public"         internal fun internalMethod() = "internal"         private fun privateMethod() = "private" } // Внутри модуля: class ModuleUser {     fun use() {        val obj = PublicClass()        obj.publicMethod()    // ✅ Public        obj.internalMethod()  // ✅ Internal (внутри модуля)        // obj.privateMethod()  // ❌ Private    } } // Из другого модуля: class ExternalUser {     fun use() {        val obj = PublicClass()        obj.publicMethod()    // ✅ Public        // obj.internalMethod()  // ❌ Internal (другой модуль)    } }`

---

## 9. `internal` и генерация байт-кода

## В字节-коде:

kotlin

`// Kotlin: @JvmInline value class InternalValue(val v: Int) internal class InternalClass { }`

java

`// В Java bytecode (упрощённо): public final class InternalValue { ... }  // public (нет модификатора) public final class InternalClass { ... }  // public (нет модификатора)`

**Важно:** `internal` **не имеет аналога в Java**, поэтому в байт-коде membros с `internal` становятся `public`, но **компилятор Kotlin** проверяет видимость.

---

## 10. `internal` vs Java `package-private`

|Характеристика|Kotlin `internal`|Java `package-private`|
|---|---|---|
|**Видимость**|Модуль (сборка)|Package|
|**Аналог**|Нет|`(пусто)`|
|**Cross-module**|❌ Нет|❌ Нет (другой package)|
|**Cross-package**|✅ Да (внутри модуля)|❌ Нет|

## Пример: Java package-private

java

`// package: com.example.core class PackagePrivate {  // package-private (нет модификатора)     void method() { } } // package: com.example.core ( тот же package) class CoreUser {     void use() {        new PackagePrivate().method();  // ✅ Тот же package    } } // package: com.example.app (другой package) class AppUser {     void use() {        // new PackagePrivate().method();  // ❌ Другой package    } }`

**Kotlin `internal`** виднее: внутри модуля **независимо от package**.

---

## 11. `internal` в Android

## Пример:_ANDROID модуль `:core:core`

kotlin

`// :core/src/com/example/core/InternalHelper.kt internal class InternalHelper {     fun help() = "help" } // :core/src/com/example/core/PublicApi.kt public class PublicApi {     internal fun internalFetch() = "internal" }`

## Пример: **Android модуль `:app`** (зависит от `:core`)

kotlin

`// :app/src/com/example/app/AppActivity.kt class AppActivity {     fun onCreate() {        // ✅ Public доступен:        val api = PublicApi()        // api.internalFetch()  // ❌ Internal НЕ доступен                 // ❌ InternalHelper НЕ доступен:        // val h = InternalHelper()    } }`

**Польза:** скрывать внутреннюю логику `:core` от `:app`.

---

## 12. Частые ошибки

## Ошибка 1: Забываешь `internal`

kotlin

`// ❌ По умолчанию = public: class InternalHelper {  // public! Видно извне модуля     fun help() = "help" } // ✅ Правильно: internal class InternalHelper {  // internal     fun help() = "help" }`

## Ошибка 2: Используй `internal` для public API

kotlin

`// ❌ Неудачно: internal class PublicApiClient {  // internal — не видно извне модуля     fun fetch() = "fetch" } // ✅ Правильно: public class PublicApiClient {  // public — видно из любых модулей     fun fetch() = "fetch" }`

## Ошибка 3: `internal` в дочернем классе

kotlin

`internal open class Base {     internal fun baseMethod() = "base" } class Derived : Base() {     fun derivedMethod() {        // ❌ НЕдоступно (другой модуль):        // baseMethod()    } }`

Если `Derived` в **другом модуле** — `internal` не доступен.

---

## 13. Когда использовать `internal`

## ✅ Используйте `internal`, если:

|Ситуация|Почему|
|---|---|
|**Внутренняя логика модуля**|Скрыть от других модулей|
|**Тестирование внутри модуля**|Тесты в том же модуле видят|
|**API библиотеки**|Public для клиентов, internal для внутри|
|**Разделение ответственности**|Internal helpers, public API|
|**Модульная архитектура**|Чёткие границы между модулями|

## ❌ Не используйте `internal`, если:

|Ситуация|Почему|
|---|---|
|**Полная скрытность**|Нужно `private`|
|**Нужно извне модуля**|Нужно `public`|
|**Только в классе**|Нужно `private`|
|**Нужно наследникам**|Нужно `protected`|

---

## 14. Краткая сводка

|Что|Как|
|---|---|
|**Объявить**|`internal class Name { }`|
|**Видимость**|Внутри модуля (сборки)|
|**Извне модуля**|❌ Не доступно|
|**Аналог в Java**|Нет (нет прямого аналога)|
|**По умолчанию**|`public` (не `internal`)|
|**Члены класса**|`internal fun method()`|
|**Генерация байт-кода**|`public` (но компилятор проверяет)|

---

## 15. Преимущества `internal`

## ✅ Преимущества:

- **Чёткие границы модуля** — скрывает внутренности
    
- **Type safety** — нельзя использовать извне
    
- **Тестирование** — тесты внутри модуля видят `internal`
    
- **API дизайн** — public для клиентов, internal для внутри
    
- **Модульность** — независимость модулей
    

## ❌ Недостатки:

- **Нет аналога в Java** — приinterop нужно `public`
    
- **Модуль = сборка** — зависит от структуры Gradle
    
- **Не для полной скрытности** — для этого `private`
    

---

**Где найти в документации:**  
[Kotlin — Visibility modifiers](https://kotlinlang.ru/docs/visibility-modifiers.html)