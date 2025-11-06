
## 1. Что такое аннотации?

Аннотации (annotations) в Java — это **метаданные**, прикреплённые к классам, методам, полям, параметрам и т.д. Они не влияют напрямую на работу кода (как ключевые слова), но дают дополнительную информацию компилятору, инструментам или рантайм-фреймворкам.

По сути, это **классы-наследники `java.lang.annotation.Annotation`**. А значит, они являются **полноценными типами** в языке.

Пример встроенной аннотации:

```java
@Override
public String toString() {
    return "Hello";
}
```

Здесь `@Override` сообщает компилятору, что метод должен переопределять метод суперкласса.


***

## 2. Где можно применять аннотации?

Аннотации можно ставить на:

* Классы, интерфейсы, enum’ы, записи
* Методы и конструкторы
* Поля и локальные переменные
* Аргументы методов
* Пакеты
* Даже **type-use аннотации** (начиная с Java 8):

```java
List<@NonNull String> names;
```


***

## 3. Как создавать собственные аннотации?

Создание своей аннотации:

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME) // где аннотация сохраняется
@Target(ElementType.METHOD)        // где её можно применять
public @interface MyAnnotation {
    String value();   // обязательный элемент
    int count() default 1; // элемент с дефолтным значением
}
```

Использование:

```java
public class Test {
    @MyAnnotation(value="hello", count=3)
    public void doSomething() {}
}
```


***

## 4. Метаданные аннотаций: `@Retention`, `@Target`, `@Documented`, `@Inherited`

* `@Retention`
  Указывает, будет ли аннотация доступна:
  * `SOURCE` — только в исходниках, отбрасывается компилятором
  * `CLASS` — попадает в .class-файл, но не доступна через reflection. Сохраняется в байткоде, но не доступна в рантайме.
  * `RUNTIME` — доступна во время выполнения (используют фреймворки)
* `@Target`
  Где можно использовать аннотацию: `METHOD`, `FIELD`, `TYPE`, `PARAMETER`, `ANNOTATION_TYPE`, `PACKAGE`, `TYPE_USE`, `TYPE_PARAMETER`
* `@Inherited`
  Если повесить на аннотацию — её можно «наследовать» при наследовании классов. Аннотация передаётся наследникам (но только если стоит на классе).
* `@Documented`
  Делает так, что аннотация попадает в Javadoc.


***

## 5. Чтение аннотаций через Reflection

Пример:

```java
Method m = Test.class.getMethod("doSomething");
MyAnnotation ann = m.getAnnotation(MyAnnotation.class);
System.out.println(ann.value());  // hello
System.out.println(ann.count());  // 3
```

То есть аннотации — это **интерфейсы с методами**, которые реализует JVM **при загрузке класса**.


***

## 6. Аннотации и обработка на этапе компиляции

С помощью **Annotation Processing Tool (APT)** можно обрабатывать аннотации **во время компиляции** и генерировать код (Lombok, MapStruct).

Можно писать свои процессоры, которые **анализируют аннотации во время компиляции** и генерируют код/проверки.

* Создаётся класс, наследующий `AbstractProcessor`
* Переопределяется метод `process(Set<? extends TypeElement>, RoundEnvironment)`
* Подключается через `META-INF/services/javax.annotation.processing.Processor`

Пример фрагмента процессора:

```java
@SupportedAnnotationTypes("MyAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class MyProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (Element element : roundEnv.getElementsAnnotatedWith(MyAnnotation.class)) {
            System.out.println("Found: " + element);
        }
        return true;
    }
}
```

⚡ Это то, как работают Lombok, MapStruct, Dagger.


***

## 7. Стереотипные и мета-аннотации

В Spring и других фреймворках аннотации **могут включать в себя другие аннотации**:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface Service {}
```

Аннотация `@Service` — это по сути `@Component`, но с дополнительной семантикой.


***

## 8. Repeatable-аннотации (Java 8+)

До Java 8 для «нескольких аннотаций» приходилось делать обёртку.

```java
public @interface Tag {
    String value();
}

// контейнер-аннотация
public @interface Tags {
    Tag[] value();
}
```

Использование (Java 7 и ниже):

```java
@Tags({
    @Tag("fast"),
    @Tag("unit-test")
})
class TestClass {}
```


С Java 8+ есть **`@Repeatable`**.

Можно объявить аннотацию как повторяемую:

```java
@Repeatable(Tags.class)
public @interface Tag {
    String value();
}

public @interface Tags {
    Tag[] value();
}
```

Использование:

```java
@Tag("fast")
@Tag("unit-test")
class TestClass {}
```

JVM при этом **всё равно хранит аннотации в контейнере `Tags`**, просто компилятор делает синтаксический сахар и автоматически упаковывает их.


***

## 9. Type Annotations и Checker Framework

Java 8 ввела аннотации на **любое использование типов**:

```java
@NonNull String s;
List<@Email String> emails;
```

Checker Framework позволяет компилятору делать статический анализ:

```java
void sendEmail(@Email String email) {...}
```


***

## 10. Продвинутые темы

* **Аннотации + Proxy / AOP**
  Spring, Hibernate, JUnit активно используют аннотации для автоматического связывания логики (например, `@Transactional` → обёртка через прокси).
* **Аннотации как DSL**
  Например, JPA (`@Entity`, `@Column`) фактически превращает код в декларативное описание схемы БД.
* **Аннотации на модульном уровне (Java 9+)**
  Можно ставить аннотации в `module-info.java`.
* **Performance**:
  * `RUNTIME`-аннотации влияют на размер class-файлов.
  * Чтение аннотаций через reflection дорого, но кешируемо.
* **Инструменты**: Lombok, MapStruct, Dagger — это всё annotation processing.


***

## 11. Почему аннотации — полноценные типы?

1. **Они компилируются в интерфейсы** (`public @interface` → `public interface extends Annotation`).
2. **Можно хранить ссылки на их `Class`** и использовать как ключи в мапах, DI-контейнерах.
3. **Можно создавать массивы аннотаций, передавать их как параметры методов**.
4. **Они участвуют в наследовании (****`@Inherited`****)**.
5. **Они могут быть generic-like (type-use annotations)**.

То есть это не «синтаксический сахар», а реально новый вид типа.


***

## 12. `@Inherited` — наследование аннотаций

Если ты ставишь аннотацию на класс, то **подкласс её НЕ наследует**.

Пример:

```java
@Retention(RetentionPolicy.RUNTIME)
@interface Role {
    String value();
}

@Role("admin")
class Base {}

class Child extends Base {}
```

Проверим через reflection:

```java
System.out.println(Base.class.getAnnotation(Role.class));  // @Role("admin")
System.out.println(Child.class.getAnnotation(Role.class)); // null !!!
```

То есть без `@Inherited` аннотации **не передаются в наследников**.


Иногда хочется, чтобы **подклассы автоматически наследовали метаданные**.

Например, в фреймворках безопасности (`@Secured`), DI (`@Singleton`), JPA (`@Entity`) это удобно.

Пример:

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Inherited
@interface Role {
    String value();
}

@Role("admin")
class Base {}

class Child extends Base {}
```

Теперь:

```java
System.out.println(Base.class.getAnnotation(Role.class));  // @Role("admin")
System.out.println(Child.class.getAnnotation(Role.class)); // @Role("admin")
```


**`@Inherited`** работает только для аннотаций на классах (не для методов, не для полей).

Поддерживается только на уровне `Class` API (`getAnnotation`), но не через `getDeclaredAnnotation`.

Если подкласс сам повесит аннотацию — она перекроет унаследованную.

```java
@Role("admin")
class Base {}

@Role("user")
class Child extends Base {}
```

`Child.class.getAnnotation(Role.class)` → `@Role("user")`.


***

## 13. getAnnotation() и getDeclaredAnnotation()


### 13.1 `getAnnotation(Class<T>)`

* Возвращает **аннотацию, если она есть у класса**
* ИЛИ (если стоит `@Inherited`) — ищет её в **иерархии родителей** до самого `Object`.
* То есть это «поиск с наследованием».

Пример:

```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@interface Role {
    String value();
}

@Role("admin")
class Base {}

class Child extends Base {}
```

Код:

```java
System.out.println(Child.class.getAnnotation(Role.class)); //@Role(value=admin)
```


### 13.2 `getDeclaredAnnotation(Class<T>)`

* Возвращает только **аннотации, которые непосредственно объявлены** на этом классе.
* Родителей иерархии не смотрит.

Тот же пример:

```java
System.out.println(Child.class.getDeclaredAnnotation(Role.class)); //null
```

Потому что `Child` сам не помечен `@Role`.


### 13.3 Обобщение

| Метод                             | Видимость             | Учитывает `@Inherited`? |
| --------------------------------- | --------------------- | ----------------------- |
| `getAnnotation(Class<T>)`         | Этот класс + родители | ✅ Да                    |
| `getDeclaredAnnotation(Class<T>)` | Только этот класс     | ❌ Нет                   |

То же самое касается массивов:

* `getAnnotations()` → все аннотации + унаследованные
* `getDeclaredAnnotations()` → только локальные


### 13.4. Почему это важно?

Фреймворки (например, Spring, JUnit) почти всегда используют `getAnnotation(...)`, потому что им нужно учитывать наследование.

А вот инструменты генерации кода (APT, Lombok) — чаще `getDeclaredAnnotation(...)`, чтобы брать только то, что реально стоит в исходнике.

