---
meta:
    title: 'Хранимые процедуры в SQL'
    description: 'Создание и использование хранимых процедур в SQL. Синтаксис, параметры, условная логика, циклы и практические примеры.'
---

# Хранимые процедуры в SQL

Хранимые процедуры — это программные блоки, которые выполняют определённую последовательность действий в базе данных.

<MySQLOnly>

В отличие от функций, процедуры могут изменять данные, выполнять сложную бизнес-логику и не обязательно возвращают значение.

</MySQLOnly>

<PostgreSQLOnly>

В отличие от функций, процедуры могут изменять данные, выполнять сложную бизнес-логику, но не могут возвращать значения.

</PostgreSQLOnly>

## Общая структура хранимой процедуры

<MySQLOnly>

```sql
CREATE PROCEDURE имя_процедуры(параметр1 ТИП, параметр2 ТИП, ...)
BEGIN
    -- логика процедуры
END;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
CREATE OR REPLACE PROCEDURE имя_процедуры(параметр1 ТИП, параметр2 ТИП, ...)
LANGUAGE plpgsql
AS $$
BEGIN
    -- логика процедуры
END;
$$;
```

`LANGUAGE plpgsql` — указывает, что процедура написана на языке **PL/pgSQL** (процедурном языке PostgreSQL).

`AS $$ ... $$` — **долларовое квотирование**, специальный способ обрамления тела процедуры. Позволяет избежать экранирования символов внутри процедуры.

</PostgreSQLOnly>

## Простой пример процедуры

Создадим процедуру для обновления информации о студенте:

<MySQLOnly>

```sql-executable-Schedule
-- Создаём процедуру
CREATE PROCEDURE update_student_info(
    IN student_id INT,
    IN new_first_name VARCHAR(50),
    IN new_last_name VARCHAR(50)
)
BEGIN
    UPDATE Student
    SET first_name = new_first_name,
        last_name = new_last_name
    WHERE id = student_id;
END;

-- Вызываем процедуру
CALL update_student_info(1, 'Alexander', 'Smirnov');

-- Проверяем результат
SELECT * FROM Student WHERE id = 1;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql-executable-Schedule
-- Создаём процедуру
CREATE OR REPLACE PROCEDURE update_student_info(
    student_id INT,
    new_first_name VARCHAR(50),
    new_last_name VARCHAR(50)
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE Student
    SET first_name = new_first_name,
        last_name = new_last_name
    WHERE id = student_id;
END;
$$;

-- Вызываем процедуру
CALL update_student_info(1, 'Alexander', 'Smirnov');

-- Проверяем результат
SELECT * FROM Student WHERE id = 1;
```

</PostgreSQLOnly>

Эта процедура принимает ID студента и новые данные, затем обновляет соответствующую запись в таблице `Student`.

<MySQLOnly>
## Типы параметров процедур

В MySQL процедуры поддерживают три типа параметров, которые можно передовать в хранимую процедуру:

-   **IN** — входные параметры (по умолчанию)
-   **OUT** — выходные параметры для возврата значений
-   **INOUT** — параметры, которые могут быть как входными, так и выходными

### Входные параметры (IN)

Входные параметры передают данные в процедуру. Это самый распространённый тип параметров:

```sql-executable-Schedule
CREATE PROCEDURE add_subject(
    IN subject_id INT,
    IN subject_name VARCHAR(100)
)
BEGIN
    INSERT INTO Subject (id, name)
    VALUES (subject_id, subject_name);
END;

-- Вызов с входными параметрами
CALL add_subject(15, 'Mathematics');
```

### Выходные параметры (OUT)

Выходные параметры позволяют процедуре возвращать значения:

```sql-executable-Schedule
CREATE PROCEDURE get_student_info(
    IN student_id INT,
    OUT student_name VARCHAR(100),
    OUT student_age INT
)
BEGIN
    SELECT
        CONCAT(first_name, ' ', last_name),
        TIMESTAMPDIFF(YEAR, birthday, CURDATE())
    INTO student_name, student_age
    FROM Student
    WHERE id = student_id;
END;

-- Вызов процедуры с выходными параметрами
CALL get_student_info(1, @name, @age);
SELECT @name AS student_name, @age AS student_age;
```


### Входные и выходные параметры (INOUT)

INOUT параметры могут принимать значение и возвращать изменённое значение:

```sql-executable-Schedule
CREATE PROCEDURE calculate_discount(
    INOUT price DECIMAL(10,2),
    IN discount_percent INT
)
BEGIN
    SET price = price - (price * discount_percent / 100);
END;

-- Использование INOUT параметра
SET @original_price = 1000.00;
CALL calculate_discount(@original_price, 15);
SELECT @original_price AS discounted_price;
```

</MySQLOnly>


<MySQLOnly>

### Пример работы трёх типов параметров

![Примеры работы параметров в хранимой процедуре](https://sql-academy.org/static/guidePage/stored-procedures/params-description.jpg 'Примеры работы параметров в хранимой процедуре')

### Ключевые различия типов параметров

| Тип параметра | Направление     | Использование                   |
| ------------- | --------------- | ------------------------------- |
| **IN**        | Входящий        | Передача данных в процедуру     |
| **OUT**       | Исходящий       | Возврат результата из процедуры |
| **INOUT**     | Двунаправленный | Изменение переданного значения  |

> **Важно:** OUT и INOUT параметры в MySQL требуют использования переменных сессии (например, `@variable_name`) при вызове процедуры.

</MySQLOnly>

## Управление хранимыми процедурами

-   **Просмотр существующих процедур**

    <MySQLOnly>

    ```sql
    SHOW PROCEDURE STATUS WHERE Db = 'your_database_name';
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    ```sql
    SELECT routine_name, routine_type
    FROM information_schema.routines
    WHERE routine_type = 'PROCEDURE' AND routine_schema = 'public';
    ```

    </PostgreSQLOnly>

-   **Удаление процедуры**

    <MySQLOnly>

    ```sql
    DROP PROCEDURE IF EXISTS add_student;
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    ```sql
    DROP PROCEDURE IF EXISTS add_student(VARCHAR, VARCHAR, DATE);
    ```

    </PostgreSQLOnly>

-   **Изменение процедуры**

    <MySQLOnly>

    Для изменения процедуры в MySQL нужно сначала удалить старую версию, а затем создать новую:

    ```sql
    DROP PROCEDURE IF EXISTS add_student;
    -- Создать новую версию процедуры
    CREATE PROCEDURE add_student(...) ...
    ```

    </MySQLOnly>

    <PostgreSQLOnly>

    В PostgreSQL можно использовать `CREATE OR REPLACE PROCEDURE`:

    ```sql
    CREATE OR REPLACE PROCEDURE add_student(
        student_first_name VARCHAR(50),
        student_last_name VARCHAR(50),
        student_birthday DATE
    )
    -- новая реализация
    ```

    </PostgreSQLOnly>

Хранимые процедуры — это мощный инструмент для реализации сложной бизнес-логики прямо в базе данных. Они обеспечивают централизацию логики, повышают производительность и гарантируют целостность данных! 🚀
