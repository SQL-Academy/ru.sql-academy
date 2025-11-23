---
meta:
  title: "Условная логика, оператор CASE"
  description: "Условная логика в SQL, использование оператор CASE WHEN THEN END"
---

# Условная логика, оператор CASE

SQL, подобно многим языкам программирования, позволяет писать условную логику, чтобы в зависимости от набора условий возвращать одно из множества возможных значений. В этой статье мы рассмотрим, как это реализуется в SQL с
помощью оператора `CASE`.

## Что такое условная логика?

Под условной логикой понимается наличие у программы нескольких путей выполнения в зависимости от каких-то условий.

Например, в базе данных «Расписание» есть таблица `Student` с полем `birthday`, отражающим дату рождения студента. Допустим,
в выборке необходимо отобразить не саму дату рождения, а текстовое значение «Совершеннолетний» или «Несовершеннолетний» в зависимости от того,
есть ли студенту 18 лет. Это и есть пример условной логики, при которой должно вывестись либо одно значение, либо другое
в зависимости от конкретного условия.

<ERD databaseName="Schedule" />

Реализация такого запроса с помощью `CASE` может выглядеть следующим образом:

<MySQLOnly>

```sql-executable-Schedule
SELECT first_name, last_name,
CASE
  WHEN TIMESTAMPDIFF(YEAR, birthday, NOW()) >= 18 THEN 'Совершеннолетний'
  ELSE 'Несовершеннолетний'
END AS status
FROM Student
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
SELECT first_name, last_name,
CASE
  WHEN EXTRACT(YEAR FROM AGE(NOW(), birthday)) >= 18 THEN 'Совершеннолетний'
  ELSE 'Несовершеннолетний'
END AS status
FROM Student
```

</PostgreSQLOnly>

## Синтаксис поискового выражения CASE

```sql
CASE
    WHEN условие_1 THEN возвращаемое_значение_1
    WHEN условие_2 THEN возвращаемое_значение_2
    WHEN условие_n THEN возвращаемое_значение_n
    [ELSE возвращаемое_значение_по_умолчанию]
END
```

Если `условие_1` возвращает истинное значение, то выражение `CASE` вернёт `возвращаемое_значение_1`, иначе будет сделана проверка
на `условие_2` и т.д. Если ни одно из предложенных условий не будет выполнено, то вернётся `NULL` или `возвращаемое_значение_по_умолчанию`,
если была использована конструкция `ELSE`.

### Пример

Рассмотрим оператор `CASE` на примере определения этапа школьного образования.

![Этапы школьного образования](https://sql-academy.org/static/guidePage/case-expression/ru_school_education_stages.png "Этапы школьного образования")

<MySQLOnly>

```sql-executable-Schedule
SELECT name,
CASE
  WHEN SUBSTRING(name, 1, INSTR(name, ' ')) IN (10, 11) THEN 'Старшая школа'
  WHEN SUBSTRING(name, 1, INSTR(name, ' ')) IN (5, 6, 7, 8, 9) THEN 'Средняя школа'
  ELSE 'Начальная школа'
END AS stage
FROM Class
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
SELECT name,
CASE
  WHEN SUBSTRING(name, 1, POSITION(' ' IN name) - 1) IN ('10', '11') THEN 'Старшая школа'
  WHEN SUBSTRING(name, 1, POSITION(' ' IN name) - 1) IN ('5', '6', '7', '8', '9') THEN 'Средняя школа'
  ELSE 'Начальная школа'
END AS stage
FROM Class
```

</PostgreSQLOnly>

<MySQLOnly>

- Сначала мы извлекаем номер класса из его названия
  ```sql
  SUBSTRING(name, 1, INSTR(name, ' '))
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Сначала мы извлекаем номер класса из его названия
  ```sql
  SUBSTRING(name, 1, POSITION(' ' IN name) - 1)
  ```

</PostgreSQLOnly>

- Далее мы проверяем вхождение данного номера в список классов, относящихся к «Старшая школа» и «Средняя школа».
- Если номер класса не находится в диапазоне 5–11, мы выводим «Начальная школа».

## Синтаксис простого выражения CASE

Оператор `CASE` также имеет и более простой синтаксис, который схож с поисковым выражением `CASE`, но
является менее гибким. Так он выглядит в общем виде:

```sql
CASE значение
    WHEN сравниваемое_значение_1 THEN возвращаемое_значение_1
    WHEN сравниваемое_значение_2 THEN возвращаемое_значение_2
    WHEN сравниваемое_значение_n THEN возвращаемое_значение_n
    [ELSE возвращаемое_значение_по_умолчанию]
END
```

В этом синтаксисе `значение` в `CASE` поочерёдно сравнивается с переданными значениями в `WHEN` и при совпадении возвращается значение, следующее за `THEN`.

Используя этот синтаксис, можно переписать наш предыдущий пример таким образом:

<MySQLOnly>

```sql-executable-Schedule
SELECT name,
CASE SUBSTRING(name, 1, INSTR(name, ' '))
  WHEN 11 THEN 'Старшая школа'
  WHEN 10 THEN 'Старшая школа'
  WHEN 9 THEN 'Средняя школа'
  WHEN 8 THEN 'Средняя школа'
  WHEN 7 THEN 'Средняя школа'
  WHEN 6 THEN 'Средняя школа'
  WHEN 5 THEN 'Средняя школа'
  ELSE 'Начальная школа'
END AS stage
FROM Class
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
SELECT name,
CASE SUBSTRING(name, 1, POSITION(' ' IN name) - 1)
  WHEN '11' THEN 'Старшая школа'
  WHEN '10' THEN 'Старшая школа'
  WHEN '9' THEN 'Средняя школа'
  WHEN '8' THEN 'Средняя школа'
  WHEN '7' THEN 'Средняя школа'
  WHEN '6' THEN 'Средняя школа'
  WHEN '5' THEN 'Средняя школа'
  ELSE 'Начальная школа'
END AS stage
FROM Class
```

</PostgreSQLOnly>

### Проверьте себя

Какое значение вернёт оператор `CASE` в данном случае?

```sql
CASE 2
  WHEN 0 THEN 'Ноль'
  WHEN 1 THEN 'Один'
  ELSE 'Много'
END
```
