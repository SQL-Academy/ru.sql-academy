---
meta:
    title: 'Хранимые функции в SQL'
    description: 'Создание и использование хранимых функций в SQL. Синтаксис, параметры, типы возвращаемых значений и практические примеры.'
---


# Хранимые функции в SQL

Хранимые функции — это мощный инструмент SQL, который позволяет создавать переиспользуемые блоки кода для выполнения вычислений и преобразования данных.
В отличие от встроенных функций, хранимые функции создаются разработчиками для решения специфических задач.

> **Хранимая функция** — это именованный блок SQL-кода, который принимает параметры, выполняет вычисления и всегда возвращает одно значение определённого типа.

## Общая структура хранимой функции

<MySQLOnly>

```sql
CREATE FUNCTION имя_функции(параметр1 ТИП, параметр2 ТИП, ...)
RETURNS тип_возвращаемого_значения
BEGIN
    -- логика функции
    RETURN результат_вычислений;
END;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
CREATE OR REPLACE FUNCTION имя_функции(параметр1 ТИП, параметр2 ТИП, ...)
RETURNS тип_возвращаемого_значения
LANGUAGE plpgsql
AS $$
BEGIN
    -- логика функции
    RETURN результат_вычислений;
END;
$$;
```

`LANGUAGE plpgsql` — указывает, что функция написана на языке **PL/pgSQL** (процедурном языке PostgreSQL).

`AS $$ ... $$` — **долларовое квотирование**, специальный способ обрамления тела функции. Позволяет избежать экранирования символов внутри функции.

</PostgreSQLOnly>

## Простой пример функции

Создадим функцию для определения, является ли человек совершеннолетним по дате рождения:

<MySQLOnly>

```sql-executable
CREATE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
END;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable
CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date)) >= 18;
END;
$$;
```

</PostgreSQLOnly>

Теперь эту функцию можно использовать в любом запросе:

<MySQLOnly>

```sql-executable
-- Создаём функцию
CREATE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
END;

-- Используем функцию
SELECT
    is_adult('2010-05-15') AS child_status,
    is_adult('2000-03-20') AS adult_status;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable
-- Создаём функцию
CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date)) >= 18;
END;
$$;

-- Используем функцию
SELECT
    is_adult('2010-05-15') AS child_status,
    is_adult('2000-03-20') AS adult_status;
```

</PostgreSQLOnly>


## Использование функций в запросах к таблицам

Хранимые функции особенно полезны при работе с реальными данными. Например, мы можем использовать нашу функцию для фильтрации студентов по возрасту:

<MySQLOnly>

```sql-executable-Schedule
-- Создаём функцию
CREATE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
END;

-- Используем функцию в запросе к таблице
SELECT
    first_name,
    last_name,
    birthday,
    is_adult(birthday) AS is_adult
FROM Student
WHERE is_adult(birthday) = TRUE
LIMIT 5;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
-- Создаём функцию
CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date)) >= 18;
END;
$$;

-- Используем функцию в запросе к таблице
SELECT
    first_name,
    last_name,
    birthday,
    is_adult(birthday) AS is_adult
FROM Student
WHERE is_adult(birthday) = TRUE
LIMIT 5;
```

</PostgreSQLOnly>

## Функции с запросами к базе данных

Хранимые функции могут выполнять SQL-запросы внутри себя для получения необходимых данных:

<MySQLOnly>

```sql-executable-Schedule
CREATE FUNCTION get_student_lessons_count(student_id INT, target_date DATE)
RETURNS INT
BEGIN
    DECLARE lessons_count INT;

    SELECT COUNT(*) INTO lessons_count
    FROM Schedule s
    INNER JOIN Student_in_class sic ON s.class = sic.class
    WHERE sic.student = student_id
      AND s.date = target_date;

    RETURN lessons_count;
END;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
CREATE OR REPLACE FUNCTION get_student_lessons_count(student_id INT, target_date DATE)
RETURNS INT
LANGUAGE plpgsql
AS $$
DECLARE
    lessons_count INT;
BEGIN
    SELECT COUNT(*) INTO lessons_count
    FROM Schedule s
    INNER JOIN Student_in_class sic ON s.class = sic.class
    WHERE sic.student = student_id
      AND s.date = target_date;

    RETURN lessons_count;
END;
$$;
```

</PostgreSQLOnly>

Эта функция подсчитывает количество уроков у конкретного студента в определённый день:

<MySQLOnly>

```sql
SELECT get_student_lessons_count(1, '2019-09-01') AS lessons_today;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
SELECT get_student_lessons_count(1, '2019-09-01') AS lessons_today;
```

</PostgreSQLOnly>


## Разбор примера с переменными

До этого мы еще не сталкивались с переменными в SQL, но это важная концепция при создании хранимых функций и процедур.
Поэтому давайте разберём предыдущий пример пошагово:

```sql
DECLARE lessons_count INT;
```

Эта строка **объявляет переменную** `lessons_count` типа `INT`. Переменная будет хранить результат нашего запроса.

<PostgreSQLOnly>

> **Важно для PostgreSQL:** Все переменные должны быть объявлены в блоке `DECLARE` до начала тела функции (до `BEGIN`). Объявлять переменные внутри тела функции нельзя.

</PostgreSQLOnly>

```sql
SELECT COUNT(*) INTO lessons_count
FROM Schedule s
INNER JOIN Student_in_class sic ON s.class = sic.class
WHERE sic.student = student_id
  AND s.date = target_date;
```

Здесь происходит **сохранение результата запроса в переменную**:

-   `SELECT COUNT(*)` — подсчитывает количество записей
-   `INTO lessons_count` — сохраняет результат в переменную `lessons_count`
-   Остальная часть — обычный SQL-запрос с JOIN и условиями

```sql
RETURN lessons_count;
```

**Возвращаем значение** переменной как результат функции.

> **Важно:** Конструкция `INTO` позволяет сохранить результат SELECT-запроса в переменную. Это основа работы с данными внутри хранимых функций.

## Управление хранимыми функциями

-   **Просмотр существующих функций**

    <MySQLOnly>

    ```sql
    SHOW FUNCTION STATUS WHERE Db = 'your_database_name';
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    ```sql
    SELECT routine_name, routine_type
    FROM information_schema.routines
    WHERE routine_type = 'FUNCTION' AND routine_schema = 'public';
    ```

    </PostgreSQLOnly>

-   **Удаление функции**

    <MySQLOnly>

    ```sql
    DROP FUNCTION IF EXISTS is_adult;
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    ```sql
    DROP FUNCTION IF EXISTS is_adult(DATE);
    ```

    </PostgreSQLOnly>

-   **Изменение функции**

    <MySQLOnly>

    Для изменения функции в MySQL нужно сначала удалить старую версию, а затем создать новую:

    ```sql
    DROP FUNCTION IF EXISTS is_adult;
    -- Создать новую версию функции
    CREATE FUNCTION is_adult(birth_date DATE) ...
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    В PostgreSQL можно использовать `CREATE OR REPLACE FUNCTION`:

    ```sql
    CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
    RETURNS BOOLEAN
    -- новая реализация
    ```

    </PostgreSQLOnly>

Хранимые функции — это мощный инструмент для создания переиспользуемой бизнес-логики прямо в базе данных. Они помогают централизовать вычисления и обеспечить консистентность данных во всём приложении! 🚀
