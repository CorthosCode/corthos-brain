
* `HAVING` — это фильтр для **групп строк**, образованных с помощью `GROUP BY`.
* Работает **после агрегации** (`COUNT`, `SUM`, `AVG`, `MAX`, `MIN` и т. д.).
* В отличие от `WHERE` (который фильтрует строки до группировки), `HAVING` применяется **к результату группировки**.


`HAVING` **всегда пишется один раз** в запросе, но **применяется ко всем группам, полученным после** `GROUP BY`.

То есть:
  1. Сначала строки объединяются в группы (`GROUP BY`).
  2. Потом каждая группа проверяется условием `HAVING`.
  3. Если группа не проходит условие — она полностью исключается из результата.


***

## Простой пример (одна группировка)

Посчитаем количество сотрудников в каждом отделе, оставим только те отделы, где их больше 5:

```sql
SELECT department_id, COUNT(*) AS num_employees
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

* `GROUP BY department_id` — разбивает строки по отделам.
* `HAVING COUNT(*) > 5` — отбирает только группы, где сотрудников больше 5.


***

## Пример с несколькими группировками

Можно группировать по **нескольким колонкам**:

```sql
SELECT department_id, job_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_id
HAVING AVG(salary) > 50000;
```

* Сначала строки группируются **по комбинации** (`department_id`, `job_id`).
* `HAVING` фильтрует такие комбинации: оставляем только те должности в отделах, где средняя зарплата > 50 000.


***

## Сравнение `WHERE` и `HAVING`

```sql
-- WHERE фильтрует строки ДО группировки
SELECT department_id, COUNT(*)
FROM employees
WHERE salary > 40000   -- исключим сотрудников с маленькой ЗП
GROUP BY department_id;

-- HAVING фильтрует группы ПОСЛЕ агрегации
SELECT department_id, COUNT(*)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 40000;  -- исключим целые отделы

```

⚡ Разница:

* `WHERE` — ограничивает входные данные.
* `HAVING` — ограничивает группы по агрегатным условиям.


***

## HAVING без GROUP BY

Да, можно! В таком случае вся таблица — это **одна группа**:

```sql
SELECT COUNT(*) AS total_employees
FROM employees
HAVING COUNT(*) > 100;
```

* Если сотрудников меньше 100, результат будет пустой.
* Если больше — вернётся число.


***

## Несколько условий в HAVING

Можно использовать `AND`, `OR`, как в `WHERE`:

```sql
SELECT department_id, COUNT(*) AS num_employees, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5
   AND AVG(salary) > 60000;
```


***

## Что можно а что нельзя

| Случай                                                              | Пример                                                                                                                            | Объяснение                                                         |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| ✅ **Агрегатная функция**                                            | `sql SELECT department_id, COUNT(*) FROM employees GROUP BY department_id HAVING COUNT(*) > 5; `                                  | Фильтруем группы по количеству строк                               |
| ✅ **Поле из&#x20;****`GROUP BY`**                                   | `sql SELECT department_id, COUNT(*) FROM employees GROUP BY department_id HAVING department_id > 50; `                            | Можно, потому что `department_id` одинаков внутри каждой группы    |
| ❌ **Поле вне&#x20;****`GROUP BY`****&#x20;и без агрегата**          | `sql SELECT department_id, COUNT(*) FROM employees GROUP BY department_id HAVING salary > 50000; `                                | `salary` не уникален в группе → SQL не знает, какое значение брать |
| ✅ **Комбинация агрегатов и группировочных полей**                   | `sql SELECT department_id, AVG(salary) FROM employees GROUP BY department_id HAVING department_id > 10 AND AVG(salary) > 60000; ` | Условие применимо: часть по `department_id`, часть по агрегату     |
| ✅ **HAVING без GROUP BY (одна виртуальная группа)**                 | `sql SELECT COUNT(*) FROM employees HAVING COUNT(*) > 0; `                                                                        | Считается, что вся таблица = одна группа                           |
| ⚠ **HAVING без агрегатов и без GROUP BY (работает непредсказуемо)** | `sql SELECT 1 HAVING 2 > 1; `                                                                                                     | Такое возможно, но используется редко. Фильтр для "одной группы"   |

