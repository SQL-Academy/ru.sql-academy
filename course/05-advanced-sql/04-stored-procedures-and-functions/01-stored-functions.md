---
meta:
    title: "Хранимые функции в SQL"
    description: "Создание и использование хранимых функций в SQL. Синтаксис, параметры, типы возвращаемых значений и практические примеры."
---

# Хранимые функции в SQL

Хранимые функции — это мощный инструмент SQL, который позволяет создавать переиспользуемые блоки кода для выполнения вычислений и преобразования данных.
В отличие от встроенных функций, хранимые функции создаются разработчиками для решения специфических задач.

> **Хранимая функция** — это именованный блок SQL-кода, который принимает параметры, выполняет вычисления и всегда возвращает одно значение определённого типа.

## Общая структура хранимой функции

**MySQL**

```sql
CREATE FUNCTION имя_функции(параметр1 ТИП, параметр2 ТИП, ...)
RETURNS тип_возвращаемого_значения
BEGIN
    -- логика функции
    RETURN результат_вычислений;
END;
```

**PostgreSQL**

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

## Простой пример функции

Создадим функцию для определения, является ли человек совершеннолетним по дате рождения:

**MySQL**

```sql
CREATE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) >= 18;
END;
```

**PostgreSQL**

```sql
CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date)) >= 18;
END;
$$;
```

Теперь эту функцию можно использовать в любом запросе:

**MySQL**

```sql
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

**PostgreSQL**

```sql
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

**MySQL**

| child_status | adult_status |
| ------------ | ------------ |
| 0            | 1            |

**PostgreSQL**

| child_status | adult_status |
| ------------ | ------------ |
| false        | true         |

## Использование функций в запросах к таблицам

Хранимые функции особенно полезны при работе с реальными данными. Например, мы можем использовать нашу функцию для фильтрации студентов по возрасту:

**MySQL**

```sql
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

**PostgreSQL**

```sql
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

**MySQL**

| first_name | last_name | birthday                 | is_adult |
| ---------- | --------- | ------------------------ | -------- |
| Nikolaj    | Sokolov   | 2000-10-01T00:00:00.000Z | 1        |
| Vyacheslav | Eliseev   | 2000-11-21T00:00:00.000Z | 1        |
| Ivan       | Efremov   | 2000-09-19T00:00:00.000Z | 1        |
| Anatolij   | ZHdanov   | 2007-07-15T00:00:00.000Z | 1        |
| Georgij    | Noskov    | 2000-03-03T00:00:00.000Z | 1        |

**PostgreSQL**

| first_name | last_name | birthday                 | is_adult |
| ---------- | --------- | ------------------------ | -------- |
| Nikolaj    | Sokolov   | 2000-10-01T00:00:00.000Z | true     |
| Vyacheslav | Eliseev   | 2000-11-21T00:00:00.000Z | true     |
| Ivan       | Efremov   | 2000-09-19T00:00:00.000Z | true     |
| Anatolij   | ZHdanov   | 2007-07-15T00:00:00.000Z | true     |
| Georgij    | Noskov    | 2000-03-03T00:00:00.000Z | true     |

## Функции с запросами к базе данных

Хранимые функции могут выполнять SQL-запросы внутри себя для получения необходимых данных:

**MySQL**

```sql
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

**PostgreSQL**

```sql
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

Эта функция подсчитывает количество уроков у конкретного студента в определённый день:

**MySQL**

```sql
SELECT get_student_lessons_count(1, '2019-09-01') AS lessons_today;
```

**PostgreSQL**

```sql
SELECT get_student_lessons_count(1, '2019-09-01') AS lessons_today;
```

| lessons_today |
| ------------- |
| 3             |

## Разбор примера с переменными

До этого мы еще не сталкивались с переменными в SQL, но это важная концепция при создании хранимых функций и процедур.
Поэтому давайте разберём предыдущий пример пошагово:

```sql
DECLARE lessons_count INT;
```

Эта строка **объявляет переменную** `lessons_count` типа `INT`. Переменная будет хранить результат нашего запроса.

**PostgreSQL**

> **Важно для PostgreSQL:** Все переменные должны быть объявлены в блоке `DECLARE` до начала тела функции (до `BEGIN`). Объявлять переменные внутри тела функции нельзя.

```sql
SELECT COUNT(*) INTO lessons_count
FROM Schedule s
INNER JOIN Student_in_class sic ON s.class = sic.class
WHERE sic.student = student_id
  AND s.date = target_date;
```

Здесь происходит **сохранение результата запроса в переменную**:

- `SELECT COUNT(*)` — подсчитывает количество записей
- `INTO lessons_count` — сохраняет результат в переменную `lessons_count`
- Остальная часть — обычный SQL-запрос с JOIN и условиями

```sql
RETURN lessons_count;
```

**Возвращаем значение** переменной как результат функции.

> **Важно:** Конструкция `INTO` позволяет сохранить результат SELECT-запроса в переменную. Это основа работы с данными внутри хранимых функций.

## Управление хранимыми функциями

- **Просмотр существующих функций**

    **MySQL**

    ```sql
    SHOW FUNCTION STATUS WHERE Db = 'your_database_name';
    ```

    **PostgreSQL**

    ```sql
    SELECT routine_name, routine_type
    FROM information_schema.routines
    WHERE routine_type = 'FUNCTION' AND routine_schema = 'public';
    ```

- **Удаление функции**

    **MySQL**

    ```sql
    DROP FUNCTION IF EXISTS is_adult;
    ```

    **PostgreSQL**

    ```sql
    DROP FUNCTION IF EXISTS is_adult(DATE);
    ```

- **Изменение функции**

    **MySQL**

    Для изменения функции в MySQL нужно сначала удалить старую версию, а затем создать новую:

    ```sql
    DROP FUNCTION IF EXISTS is_adult;
    -- Создать новую версию функции
    CREATE FUNCTION is_adult(birth_date DATE) ...
    ```

    **PostgreSQL**

    В PostgreSQL можно использовать `CREATE OR REPLACE FUNCTION`:

    ```sql
    CREATE OR REPLACE FUNCTION is_adult(birth_date DATE)
    RETURNS BOOLEAN
    -- новая реализация
    ```

Хранимые функции — это мощный инструмент для создания переиспользуемой бизнес-логики прямо в базе данных. Они помогают централизовать вычисления и обеспечить консистентность данных во всём приложении! 🚀
