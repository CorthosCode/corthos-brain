`@JvmOverloads` — это аннотация Kotlin, которая заставляет компилятор генерировать **перегруженные методы (overloads)** для функций/конструкторов с **значениями по умолчанию**, чтобы их было удобно вызывать из Java.

👉 Проблема:  
Kotlin поддерживает default arguments, Java — **нет**.

---

## ⚙️ Как работает

Kotlin-код:

```java
fun log(message: String, level: Int = 1, tag: String = "DEFAULT")
```


Без `@JvmOverloads`
Java увидит только:

```java
log(String message, int level, String tag)
```


👉 И придется всегда передавать ВСЕ параметры.


---

## С `@JvmOverloads`:

```java
@JvmOverloads  
fun log(message: String, level: Int = 1, tag: String = "DEFAULT")
```

👉 Сгенерируется:

```java
log(String message, int level, String tag)  
log(String message, int level)  
log(String message)
```


---

## 🧠 Правило генерации overload'ов

Генерация идет **справа налево**, пока есть default значения.

```java
fun f(a: Int, b: Int = 2, c: Int = 3)
```


👉 Генерация:

```java
f(int a, int b, int c)  
f(int a, int b)  
f(int a)
```

❗ Но НЕ будет:

```java
f(int a, int c) ❌
```


---

## 📍 Где можно использовать

1. **Функции**

```java
@JvmOverloads  
fun connect(host: String, port: Int = 80)
```

2. **Конструкторы**

```java
class ApiClient @JvmOverloads constructor(  
    val host: String,  
    val port: Int = 80  
)
```

3. **Методы в `object` / `companion object`**

⚠️ Важно:

```java
object Utils {  
    @JvmStatic  
    @JvmOverloads  
    fun foo(x: Int = 10) {}  
}
```

👉 Без `@JvmStatic`:

```java
Utils.INSTANCE.foo()
```

👉 С `@JvmStatic`:

```java
Utils.foo()
```

