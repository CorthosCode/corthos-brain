
* Интерфейс из **JPA (****`jakarta.persistence`****)**, который описывает стандартный контракт работы с ORM.
* Hibernate реализует `EntityManager` внутри себя через обёртку над `Session`.
* То есть, если ты используешь JPA, то пишешь код под `EntityManager`, а Hibernate внутри всё равно работает через `Session`.


***

## Сравнение Hibernate API vs JPA API

Hibernate

```java
SessionFactory factory = new Configuration()
        .configure("hibernate.cfg.xml")
        .addAnnotatedClass(Employee.class)
        .buildSessionFactory();

Session session = factory.openSession();
Transaction tx = session.beginTransaction();

List<Employee> list = session.createQuery("from Employee", Employee.class).list();

tx.commit();
session.close();
```

JPA (через `EntityManager`):

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPersistenceUnit");
EntityManager em = emf.createEntityManager();

em.getTransaction().begin();

List<Employee> list = em.createQuery("from Employee", Employee.class).getResultList();

em.getTransaction().commit();
em.close();
```

👉 Отличие:

* `SessionFactory` → `EntityManagerFactory`
* `Session` → `EntityManager`
* Транзакции (`beginTransaction`, `commit`) работают очень похоже
* Запросы (`createQuery`) выглядят почти одинаково (только методы другие: `getResultList`, `getSingleResult`)


***

## Зачем нужен `EntityManager`, если есть `Session`

**Стандарт**

Код под JPA (`EntityManager`) будет работать и на Hibernate, и на EclipseLink, и на OpenJPA.

Код под `Session` — только на Hibernate.


**Совместимость со Spring**

Spring Data JPA работает именно через `EntityManager`.

Поэтому если ты хочешь потом мигрировать к Spring Data — лучше привыкать к `EntityManager`.


**Упрощённый API**

У `EntityManager` есть простые методы:

* `find()` вместо `session.get()`
* `persist()` вместо `session.persist()`
* `merge()` вместо `session.merge()`
* `remove()` вместо `session.delete()`


***

## Примеры методов `EntityManager`

```java
Employee e = new Employee();
e.setFullName("Petrov");

// persist → INSERT
em.persist(e);

// find → SELECT by id
Employee emp = em.find(Employee.class, 1L);

// merge → UPDATE (или INSERT если нет)
emp.setFullName("Ivanov");
em.merge(emp);

// remove → DELETE
em.remove(emp);

// JPQL запрос
List<Employee> list = em.createQuery(
        "select e from Employee e where e.fullName like :name", Employee.class)
    .setParameter("name", "P%")
    .getResultList();
```

