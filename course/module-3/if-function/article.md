---
meta:
  title: "Условная логика в SQL, функции IFNULL, NULLIF"
  description: "Условная логика в SQL: функция IF в MySQL и дополнительные функции в PostgreSQL (COALESCE, NULLIF)"
---

<MySQLOnly>

# Условная функция IF

В предыдущем уроке мы рассматривали оператор `CASE` для реализации условной логики в SQL.
Однако это не единственный механизм, с помощью которого возможно реализовать ветвление логики в запросе.
Пришло время обратить наше внимание на функцию `IF`.

## Синтаксис IF

```sql
IF(условное_выражение, значение_1, значение_2);
```

Если условное выражение, передаваемое в качестве первого аргумента в функцию `IF`,
истинно, функция вернёт значение второго аргумента `значение_1`, иначе возвращается значение третьего аргумента `значение_2`.

</MySQLOnly>

<PostgreSQLOnly>

# Дополнительные функции условной логики

В предыдущем уроке мы изучили оператор `CASE` для реализации условной логики в SQL.
PostgreSQL предоставляет дополнительные функции, которые упрощают работу с условной логикой в специальных случаях.
Эти функции особенно полезны при работе с `NULL` значениями и создании более читаемого кода.

## Функции для условной логики

Помимо универсального оператора `CASE`, PostgreSQL предоставляет:

1. **Функция COALESCE** - для работы с `NULL` значениями
2. **Функция NULLIF** - для специальных случаев с `NULL`

Эти функции являются стандартными SQL функциями и делают код более читаемым в определенных ситуациях.

</PostgreSQLOnly>

### Примеры

<MySQLOnly>

- Простое сравнение двух чисел. Так как 10 не больше 20, функция вернёт 'FALSE'.

  ```sql-executable-Airbnb
  SELECT IF(10 > 20, 'TRUE', 'FALSE');
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Простой пример условной логики с помощью оператора CASE из предыдущего урока:

  ```sql-executable-Airbnb
  SELECT CASE WHEN 10 > 20 THEN 'TRUE' ELSE 'FALSE' END;
  ```

</PostgreSQLOnly>

<MySQLOnly>

- Пример использования с реальной БД. Необходимо на основании цены определить принадлежность жилья к одному из двух классов: "Комфорт-класс" и "Эконом-класс".
  Если цена больше или равна `150`, то это жильё относится к "Комфорт-класс".

  <ERD databaseName="Airbnb" />

  ```sql-executable-Airbnb
  SELECT id, price,
      IF(price >= 150, 'Комфорт-класс', 'Эконом-класс') AS category
      FROM Rooms
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Пример с реальными данными. Оператор CASE помогает категоризировать жильё по цене:

  <ERD databaseName="Airbnb" />

  ```sql-executable-Airbnb
  SELECT id, price,
      CASE WHEN price >= 150 THEN 'Комфорт-класс' ELSE 'Эконом-класс' END AS category
      FROM Rooms
  ```

</PostgreSQLOnly>

<MySQLOnly>

- Функции `IF` можно также вкладывать друг в друга, эмулируя оператор `CASE`.

  ```sql-executable-Airbnb
  SELECT id, price,
      IF(price >= 200, 'Бизнес-класс',
          IF(price >= 150,
              'Комфорт-класс', 'Эконом-класс')) AS category
      FROM Rooms
  ```

</MySQLOnly>

<PostgreSQLOnly>

- CASE отлично подходит для множественных условий:

  ```sql-executable-Airbnb
  SELECT id, price,
      CASE
          WHEN price >= 200 THEN 'Бизнес-класс'
          WHEN price >= 150 THEN 'Комфорт-класс'
          ELSE 'Эконом-класс'
      END AS category
      FROM Rooms
  ```

Однако для специальных случаев PostgreSQL предоставляет более специализированные функции.

</PostgreSQLOnly>

<MySQLOnly>

## Функции IFNULL и NULLIF

Помимо функции `IF`, в MySQL также есть более простые, но менее универсальные функции `IFNULL` и `NULLIF`,
направленные на обработку `NULL` значений.

### Синтаксис IFNULL

```sql
IFNULL(значение, альтернативное_значение);
```

Функция `IFNULL` возвращает `значение`, переданное первым аргументом, если оно не равно `NULL`, иначе возвращает
`альтернативное_значение`.

</MySQLOnly>

<PostgreSQLOnly>

## Функция COALESCE

Функция `COALESCE` - это элегантное решение для работы с `NULL` значениями.
Она возвращает первое не-NULL значение из списка аргументов.

### Синтаксис

```sql
COALESCE(значение1, значение2, ..., значениеN);
```

Это намного удобнее, чем писать длинные CASE выражения для обработки `NULL`.

### Сравнение подходов

С помощью CASE:

```sql
CASE
    WHEN значение1 IS NOT NULL THEN значение1
    WHEN значение2 IS NOT NULL THEN значение2
    ELSE значение3
END
```

С помощью COALESCE (намного проще):

```sql
COALESCE(значение1, значение2, значение3)
```

</PostgreSQLOnly>

<MySQLOnly>
### Примеры с функцией IFNULL

- Если первый аргумент не равен `NULL`, то вернётся именно он.

  ```sql-executable-Airbnb
  SELECT IFNULL('SQL Academy', 'Альтернатива SQL Academy') AS sql_trainer;
  ```

</MySQLOnly>

<PostgreSQLOnly>
### Примеры с функцией COALESCE

- Если первый аргумент не равен `NULL`, то вернётся именно он.

  ```sql-executable-Airbnb
  SELECT COALESCE('SQL Academy', 'Альтернатива SQL Academy') AS sql_trainer;
  ```

</PostgreSQLOnly>

<MySQLOnly>

- Если первый аргумент равен `NULL`, то вернётся значение, переданное вторым аргументом.

  ```sql-executable-Airbnb
  SELECT IFNULL(NULL, 'Альтернатива SQL Academy') AS sql_trainer;
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Если первый аргумент равен `NULL`, то вернётся следующее не-NULL значение.

  ```sql-executable-Airbnb
  SELECT COALESCE(NULL, 'Альтернатива SQL Academy') AS sql_trainer;
  ```

- `COALESCE` может принимать множество аргументов, что делает код очень читаемым:

  ```sql-executable-Airbnb
  SELECT COALESCE(NULL, NULL, 'SQL Academy', 'Запасной вариант') AS sql_trainer;
  ```

## Функция NULLIF

Функция `NULLIF` полезна, когда нужно заменить определенное значение на NULL.
Это может пригодиться для фильтрации или обработки "пустых" значений.

</PostgreSQLOnly>

### Синтаксис NULLIF

```sql
NULLIF(значение_1, значение_2);
```

Функция `NULLIF` возвращает `NULL`, если `значение_1` равно `значению_2`, в противном случае возвращает `значение_1`.

### Примеры с функцией NULLIF

<MySQLOnly>

- Если значение первого аргумента равно значению второго аргумента, то возвращается `NULL`.

  ```sql-executable-Airbnb
  SELECT NULLIF('SQL Academy', 'SQL Academy') AS sql_trainer;
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Если значение первого аргумента равно значению второго аргумента, то возвращается `NULL`.

  ```sql-executable-Airbnb
  SELECT NULLIF('SQL Academy', 'SQL Academy') AS sql_trainer;
  ```

</PostgreSQLOnly>

<MySQLOnly>

- Если значения первого и второго аргумента различаются, то возвращается значение первого аргумента.

  ```sql-executable-Airbnb
  SELECT NULLIF('SQL Academy', 'Альтернатива SQL Academy') AS sql_trainer;
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Если значения первого и второго аргумента различаются, то возвращается значение первого аргумента.

  ```sql-executable-Airbnb
  SELECT NULLIF('SQL Academy', 'Альтернатива SQL Academy') AS sql_trainer;
  ```

</PostgreSQLOnly>

<PostgreSQLOnly>

### Когда использовать каждую функцию:

- **CASE**: Когда нужна сложная условная логика с множественными условиями
- **COALESCE**: Когда нужно заменить NULL значения на значения по умолчанию
- **NULLIF**: Когда нужно превратить определенные значения в NULL

Эти функции делают код более читаемым и являются частью стандарта SQL.

</PostgreSQLOnly>
