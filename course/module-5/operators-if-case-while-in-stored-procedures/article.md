---
meta:
    title: 'Операторы IF, CASE, WHILE в хранимых процедурах'
    description: 'Изучите операторы ветвления и циклы в хранимых процедурах SQL. Синтаксис и примеры IF, CASE, WHILE для MySQL и PostgreSQL.'
---

# Операторы IF, CASE, WHILE в хранимых процедурах

Хранимые процедуры и функции — это не просто удобные контейнеры для группы запросов. Они позволяют реализовать достаточно сложную логику, используя операторы ветвления и циклы.

В этой статье мы рассмотрим основные операторы управления потоком выполнения: условные операторы `IF` и `CASE`, а также циклы `WHILE`.

## Условный оператор IF

Оператор `IF` позволяет выполнять код в зависимости от выполнения условия.

### Синтаксис IF

<MySQLOnly>

```sql
IF условие THEN
    -- код, выполняемый при истинном условии
ELSEIF другое_условие THEN
    -- код для альтернативного условия
ELSE
    -- код по умолчанию
END IF;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
IF условие THEN
    -- код, выполняемый при истинном условии
ELSIF другое_условие THEN
    -- код для альтернативного условия
ELSE
    -- код по умолчанию
END IF;
```

</PostgreSQLOnly>

### Пример использования IF

Создадим процедуру, которая определяет категорию студента по возрасту:

<MySQLOnly>

```sql-executable-Schedule
CREATE PROCEDURE categorize_student_by_age(
    IN student_id INT,
    OUT category VARCHAR(20)
)
BEGIN
    DECLARE student_age INT;

    -- Получаем возраст студента
    SELECT TIMESTAMPDIFF(YEAR, birthday, CURDATE())
    INTO student_age
    FROM Student
    WHERE id = student_id;

    -- Определяем категорию по возрасту
    IF student_age < 18 THEN
        SET category = 'Несовершеннолетний';
    ELSEIF student_age BETWEEN 18 AND 25 THEN
        SET category = 'Молодой';
    ELSE
        SET category = 'Взрослый';
    END IF;
END;

-- Использование процедуры
CALL categorize_student_by_age(1, @category);
SELECT @category AS age_category;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
CREATE OR REPLACE FUNCTION categorize_student_by_age(student_id INT)
RETURNS VARCHAR(20)
LANGUAGE plpgsql
AS $$
DECLARE
    student_age INT;
    category VARCHAR(20);
BEGIN
    -- Получаем возраст студента
    SELECT EXTRACT(YEAR FROM AGE(CURRENT_DATE, birthday))
    INTO student_age
    FROM Student
    WHERE id = student_id;

    -- Определяем категорию по возрасту
    IF student_age < 18 THEN
        category := 'Несовершеннолетний';
    ELSIF student_age BETWEEN 18 AND 25 THEN
        category := 'Молодой';
    ELSE
        category := 'Взрослый';
    END IF;

    RETURN category;
END;
$$;

-- Использование функции
SELECT categorize_student_by_age(1) AS age_category;
```

</PostgreSQLOnly>

## Оператор выбора CASE

Оператор `CASE` предоставляет более элегантный способ обработки множественных условий.

### Синтаксис CASE

<MySQLOnly>

```sql
CASE
    WHEN условие1 THEN результат1
    WHEN условие2 THEN результат2
    ELSE результат_по_умолчанию
END CASE;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
CASE
    WHEN условие1 THEN результат1
    WHEN условие2 THEN результат2
    ELSE результат_по_умолчанию
END CASE;
```

</PostgreSQLOnly>

### Пример использования CASE

Создадим ту же процедуру категоризации студентов, но используя оператор CASE:

<MySQLOnly>

```sql-executable-Schedule
CREATE PROCEDURE categorize_student_with_case(
    IN student_id INT,
    OUT category VARCHAR(20)
)
BEGIN
    DECLARE student_age INT;

    -- Получаем возраст студента
    SELECT TIMESTAMPDIFF(YEAR, birthday, CURDATE())
    INTO student_age
    FROM Student
    WHERE id = student_id;

    -- Определяем категорию с помощью CASE
    SET category = CASE
        WHEN student_age < 18 THEN 'Несовершеннолетний'
        WHEN student_age BETWEEN 18 AND 25 THEN 'Молодой'
        ELSE 'Взрослый'
    END;
END;

-- Использование процедуры
CALL categorize_student_with_case(1, @category);
SELECT @category AS age_category;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
CREATE OR REPLACE FUNCTION categorize_student_with_case(student_id INT)
RETURNS VARCHAR(20)
LANGUAGE plpgsql
AS $$
DECLARE
    student_age INT;
    category VARCHAR(20);
BEGIN
    -- Получаем возраст студента
    SELECT EXTRACT(YEAR FROM AGE(CURRENT_DATE, birthday))
    INTO student_age
    FROM Student
    WHERE id = student_id;

    -- Определяем категорию с помощью CASE
    category := CASE
        WHEN student_age < 18 THEN 'Несовершеннолетний'
        WHEN student_age BETWEEN 18 AND 25 THEN 'Молодой'
        ELSE 'Взрослый'
    END;

    RETURN category;
END;
$$;

-- Использование функции
SELECT categorize_student_with_case(1) AS age_category;
```

</PostgreSQLOnly>

## Цикл WHILE

Цикл `WHILE` позволяет выполнять код повторно, пока выполняется определённое условие.

### Синтаксис WHILE

<MySQLOnly>

```sql
WHILE условие DO
    -- код, выполняемый в цикле
END WHILE;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
WHILE условие LOOP
    -- код, выполняемый в цикле
END LOOP;
```

</PostgreSQLOnly>

### Пример использования WHILE

Создадим процедуру для создания нескольких тестовых предметов:

<MySQLOnly>

```sql-executable-Schedule
CREATE PROCEDURE create_test_subjects(IN count_subjects INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE subject_id INT DEFAULT 20;

    WHILE i <= count_subjects DO
        INSERT INTO Subject (id, name)
        VALUES
        (
            subject_id + i,
            CONCAT('Test Subject ', i)
        );

        SET i = i + 1;
    END WHILE;
END;

-- Создаём 3 тестовых предмета
CALL create_test_subjects(3);

-- Проверяем результат
SELECT * FROM Subject WHERE name LIKE 'Test Subject%';
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
CREATE OR REPLACE PROCEDURE create_test_subjects(count_subjects INT)
LANGUAGE plpgsql
AS $$
DECLARE
    i INT := 1;
    subject_id INT := 20;
BEGIN
    WHILE i <= count_subjects LOOP
        INSERT INTO Subject (id, name)
        VALUES
        (
            subject_id + i,
            'Test Subject ' || i
        );

        i := i + 1;
    END LOOP;

    RAISE NOTICE 'Создано % тестовых предметов', count_subjects;
END;
$$;

-- Создаём 3 тестовых предмета
CALL create_test_subjects(3);

-- Проверяем результат
SELECT * FROM Subject WHERE name LIKE 'Test Subject%';
```

</PostgreSQLOnly>


Операторы управления потоком делают хранимые процедуры и функции мощным инструментом для реализации сложной бизнес-логики прямо в базе данных! 🚀
