---
meta:
    title: "Операторы IF, CASE, WHILE в хранимых процедурах и функциях"
    description: "Изучите операторы ветвления и циклы в хранимых процедурах и функциях SQL. Синтаксис и примеры IF, CASE, WHILE для MySQL и PostgreSQL."
---

**MySQL**

# Операторы IF, CASE, WHILE в хранимых процедурах

**PostgreSQL**

# Операторы IF, CASE, WHILE в хранимых функциях

Хранимые процедуры и функции — это не просто удобные контейнеры для группы запросов. Они позволяют реализовать достаточно сложную логику, используя операторы ветвления и циклы.

В этой статье мы рассмотрим основные операторы управления потоком выполнения: условные операторы `IF` и `CASE`, а также циклы `WHILE`.

## Условный оператор IF

Оператор `IF` позволяет выполнять код в зависимости от выполнения условия.

### Синтаксис IF

**MySQL**

```sql
IF условие THEN
    действия
[ELSEIF условие THEN
    действия] ...
[ELSE
    действия]
END IF;
```

- `условие` — логическое выражение, например `student_age < 18`. Срабатывает первая ветка, чьё условие
  истинно, остальные пропускаются.
- `действия` — один или несколько SQL-операторов, каждый заканчивается `;`. Выполняются, если выбрана
  эта ветка.
- `ELSEIF` (ключевое слово, необязательный) — ещё одна ветка со своим условием. Проверяется, только если все условия выше ложны.
  Таких веток может быть несколько.
- `ELSE` (ключевое слово, необязательный) — ветка, которая выполнится, если ни одно условие не оказалось истинным.

**PostgreSQL**

```sql
IF условие THEN
    действия
[ELSIF условие THEN
    действия] ...
[ELSE
    действия]
END IF;
```

- `условие` — логическое выражение, например `student_age < 18`. Срабатывает первая ветка, чьё условие
  истинно, остальные пропускаются.
- `действия` — один или несколько SQL-операторов, каждый заканчивается `;`. Выполняются, если выбрана
  эта ветка.
- `ELSIF` (ключевое слово, необязательный) — ещё одна ветка со своим условием. Проверяется, только если все условия выше ложны.
  Таких веток может быть несколько.
- `ELSE` (ключевое слово, необязательный) — ветка, которая выполнится, если ни одно условие не оказалось истинным.

### Пример использования IF

**MySQL**

Создадим процедуру, которая определяет категорию студента по возрасту:

```sql
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

**PostgreSQL**

Создадим функцию, которая определяет категорию студента по возрасту:

```sql
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

**MySQL**

| age_category |
| ------------ |
| Молодой      |

## Оператор выбора CASE

Оператор `CASE` предоставляет более элегантный способ обработки множественных условий.

### Синтаксис CASE

**MySQL**

```sql
CASE
    WHEN условие THEN результат
    [WHEN ...]
    [ELSE результат_по_умолчанию]
END
```

- `условие` — логическое выражение, которое проверяется в своей ветке `WHEN`. Веток может быть
  несколько: срабатывает первая, чьё условие истинно.
- `результат` — значение, которое получит `CASE`, если условие этой ветки истинно.
- `результат_по_умолчанию` (необязательный) — значение на случай, когда ни одно условие не выполнено. Без `ELSE` вернётся `NULL`.

**PostgreSQL**

```sql
CASE
    WHEN условие THEN результат
    [WHEN ...]
    [ELSE результат_по_умолчанию]
END
```

- `условие` — логическое выражение, которое проверяется в своей ветке `WHEN`. Веток может быть
  несколько: срабатывает первая, чьё условие истинно.
- `результат` — значение, которое получит `CASE`, если условие этой ветки истинно.
- `результат_по_умолчанию` (необязательный) — значение на случай, когда ни одно условие не выполнено. Без `ELSE` вернётся `NULL`.

### Пример использования CASE

**MySQL**

Создадим ту же процедуру категоризации студентов, но используя оператор CASE:

```sql
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

**PostgreSQL**

Создадим ту же функцию категоризации студентов, но используя оператор CASE:

```sql
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

**MySQL**

| age_category |
| ------------ |
| Молодой      |

## Цикл WHILE

Цикл `WHILE` позволяет выполнять код повторно, пока выполняется определённое условие.

### Синтаксис WHILE

**MySQL**

```sql
WHILE условие DO
    действия
END WHILE;
```

- `условие` — логическое выражение, которое проверяется перед каждым проходом цикла. Пока оно
  истинно, действия повторяются.
- `действия` — SQL-операторы тела цикла. Среди них должно быть что-то, что со временем сделает
  условие ложным, иначе цикл не закончится.

**PostgreSQL**

```sql
WHILE условие LOOP
    действия
END LOOP;
```

- `условие` — логическое выражение, которое проверяется перед каждым проходом цикла. Пока оно
  истинно, действия повторяются.
- `действия` — SQL-операторы тела цикла. Среди них должно быть что-то, что со временем сделает
  условие ложным, иначе цикл не закончится.

### Пример использования WHILE

Рассмотрим пример хранимой процедуры для создания нескольких тестовых предметов:

**MySQL**

```sql
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

**PostgreSQL**

```sql
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

| id  | name           |
| --- | -------------- |
| 21  | Test Subject 1 |
| 22  | Test Subject 2 |
| 23  | Test Subject 3 |

Операторы управления потоком делают хранимые процедуры и функции мощным инструментом для реализации сложной бизнес-логики прямо в базе данных! 🚀
