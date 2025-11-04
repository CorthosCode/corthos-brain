# 📒 Глава 5: JPQL и HQL

Hibernate позволяет работать с БД не только через `session.get()` или `find()`, но и писать **объектно-ориентированные запросы**, которые преобразуются в SQL. Это даёт гибкость и переносимость.





***

## Что такое HQL и JPQL?

**HQL (Hibernate Query Language)** — «родной» язык Hibernate.

**JPQL (Java Persistence Query Language)** — стандарт JPA, почти полностью совпадает с HQL.

👉 Отличие:

* JPQL = стандарт → работает с любой JPA-имплементацией.
* HQL = расширения от Hibernate (более мощный, но привязан к Hibernate).





***

## Базовый синтаксис

Пишем запросы не по таблицам, а по **сущностям и их полям**.

```java
// Все сотрудники
List<Employee> list = session.createQuery("from Employee", Employee.class).list();

// С условием
Employee e = session.createQuery(
    "from Employee e where e.fullName = :name", Employee.class)
    .setParameter("name", "Petrov")
    .uniqueResult();
```

👉 Здесь `Employee` = имя сущности (`@Entity`), а не таблицы.





***

## SELECT

```java
// выбор конкретных полей
List<String> names = session.createQuery(
    "select e.fullName from Employee e", String.class).list();

// проекция в DTO
List<EmployeeDTO> dtos = session.createQuery(
    "select new com.example.EmployeeDTO(e.id, e.fullName) from Employee e",
    EmployeeDTO.class
).list();
```





***

## JOIN FETCH

По умолчанию ассоциации `@OneToMany` и `@ManyToOne` часто ленивые (`LAZY`).

Hibernate создаёт прокси: при первом доступе → делает отдельный SQL (проблема **N+1**).

```java
// без join fetch
List<Department> deps = session.createQuery("from Department", Department.class).list();
for (Department d : deps) {
    System.out.println(d.getEmployees().size()); // каждый раз отдельный select
}
```

➡ Решение: `JOIN FETCH`.

```java
List<Department> deps = session.createQuery(
    "select distinct d from Department d join fetch d.employees", Department.class
).list();
```

* Hibernate сделал **один SQL** с join-ом:

```sql
select d.*, e.* 
from department d 
join employee e on d.id = e.department_id;
```

Вернул департаменты **с уже загруженными** сотрудниками.



**Можно ли использовать нативный SQL?**

Да, **можно**. Hibernate не запрещает.

```java
List<Object[]> rows = session.createNativeQuery(
    "select d.*, e.* from department d join employee e on d.id = e.department_id"
).list();
```

Но:

* вернётся `List<Object[]>` (или «сырые данные»), а не `List<Department>`.
* тебе самому нужно будет маппить результат → сущности (`ResultSet → entity`).
* теряется главное преимущество ORM: объектная работа без ручного преобразования.





***

## Параметры

```java
// позиционные
query.setParameter(1, "IT");

// именованные
query.setParameter("name", "Petrov");
```





***

## Агрегации и группировка

```java
// количество сотрудников в департаменте
List<Object[]> stats = session.createQuery(
    "select d.name, count(e) from Department d join d.employees e group by d.name"
).list();

for (Object[] row : stats) {
    System.out.println(row[0] + " -> " + row[1]);
}
```





***

## Подзапросы

```java
// сотрудники из департамента "IT"
List<Employee> list = session.createQuery(
    "from Employee e where e.department.id = " +
    "(select d.id from Department d where d.name = :depName)", Employee.class
)
.setParameter("depName", "IT")
.list();
```





***

## Update / Delete (bulk operations)

⚠️ Bulk-операции **не синхронизируют Persistence Context** (кеш сессии).

Нужно делать `session.clear()` после них.

```java
// массовое обновление
int updated = session.createQuery(
    "update Employee e set e.fullName = :newName where e.fullName = :oldName")
    .setParameter("newName", "Ivanov")
    .setParameter("oldName", "Petrov")
    .executeUpdate();

// массовое удаление
int deleted = session.createQuery(
    "delete from Employee e where e.department.name = :depName")
    .setParameter("depName", "IT")
    .executeUpdate();
```





***

## Native Query

Если чего-то нет в JPQL/HQL, можно писать чистый SQL:

```java
List<Employee> list = session.createNativeQuery(
    "select * from employee where full_name like :name", Employee.class)
    .setParameter("name", "Pet%")
    .list();
```



