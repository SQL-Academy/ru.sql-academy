---
meta:
  title: "Создание и удаление таблиц: MySQL и PostgreSQL"
  description: "SQL создание и удаление таблиц в MySQL и PostgreSQL. Операторы описания таблиц."
---

# Создание и удаление таблиц

## Создание таблицы

<MySQLOnly>

Перед созданием таблицы необходимо выбрать базу данных, в которую таблица будет записана. Это делается с помощью оператора `USE`:

```sql
USE имя_базы_данных;
```

</MySQLOnly>

Для создания таблицы используется оператор `CREATE TABLE`. Его базовый синтаксис имеет следующий вид:

```sql
CREATE TABLE [IF NOT EXISTS] имя_таблицы (
     столбец_1 тип_данных,
    [столбец_2 тип_данных,]
    ...
    [столбец_n тип_данных,]
);
```

Например, создадим таблицу пользователей.

<MySQLOnly>

```sql
CREATE TABLE Users (
    id INT,
    name VARCHAR(255),
    age INT
);
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
CREATE TABLE Users (
    id INTEGER,
    name VARCHAR(255),
    age INTEGER
);
```

</PostgreSQLOnly>

<MySQLOnly>

`INT`, `VARCHAR(255)` - типы данных: числовой и строковый соответственно. Более подробно о них можно будет узнать в следующих статьях.

</MySQLOnly>

<PostgreSQLOnly>

`INTEGER`, `VARCHAR(255)` - типы данных: числовой и строковый соответственно. Более подробно о них можно будет узнать в следующих статьях.

</PostgreSQLOnly>

## Дополнительные параметры определения столбцов

Вышеприведённое определение столбцов в таблице является упрощённым.
Помимо названия столбца и его типа в определение иногда необходимо добавлять следующие необязательные параметры:

- `PRIMARY KEY`

  Указывает колонку или множество колонок как первичный ключ.

<MySQLOnly>

- `AUTO_INCREMENT`

  Указывает, что значение данной колонки будет автоматически увеличиваться при добавлении новых записей в таблицу. Каждая таблица имеет максимум одну `AUTO_INCREMENT` колонку.
  Стоит отметить, что данный параметр можно применять только к целочисленным типам и к типам с плавающей запятой.

</MySQLOnly>

<PostgreSQLOnly>

- `SERIAL` или `GENERATED ALWAYS AS IDENTITY`

  Указывает, что значение данной колонки будет автоматически увеличиваться при добавлении новых записей в таблицу. `SERIAL` — это сокращение для создания автоинкрементного поля.

</PostgreSQLOnly>

- `UNIQUE`

  Указывает, что значения в данной колонке для всех записей должны быть отличными друг от друга.

- `NOT NULL`

  Указывает, что значения в данной колонке должны быть отличными от `NULL`.

- `DEFAULT`

  Указывает значение по умолчанию.

Для нашей таблицы пользователей можно указать следующие параметры:

<MySQLOnly>

```sql
CREATE TABLE Users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    age INT NOT NULL DEFAULT 18
);
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    age INTEGER NOT NULL DEFAULT 18
);
```

</PostgreSQLOnly>

Так, в данном примере:

<MySQLOnly>

- `id` - поле числового типа, являющееся первичным ключом с автоинкрементом;

</MySQLOnly>

<PostgreSQLOnly>

- `id` - поле типа SERIAL (автоинкрементное целое число), являющееся первичным ключом;

</PostgreSQLOnly>

- `name` - поле строкового типа с максимальной длиной в 255 символов, являющееся обязательным к заполнению;
- `age` - поле числового типа со значением по умолчанию равным 18.

## Описание таблицы

<MySQLOnly>

Для того, чтобы посмотреть описание созданной таблицы, можно воспользоваться оператором `DESCRIBE`.

```sql
DESCRIBE Users;
```

</MySQLOnly>

<PostgreSQLOnly>

Для того, чтобы посмотреть описание созданной таблицы, можно воспользоваться SQL запросом к информационной схеме:

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = 'users';
```

</PostgreSQLOnly>

## Дополнительные параметры определения таблицы

Помимо описания столбцов, при создании таблицы можно дополнительно указать следующие параметры:

<MySQLOnly>

- Первичный ключ.

  Если вы не определили первичный ключ с помощью параметров столбца, то это можно сделать с помощью дополнительных параметров таблицы, добавив запись `PRIMARY KEY (<столбец_1>, <столбец_n>)` после перечисления столбцов:

  ```sql
  CREATE TABLE Users (
      id INT,
      name VARCHAR(255) NOT NULL,
      age INT NOT NULL DEFAULT 18,
      PRIMARY KEY (id)
  );
  ```

</MySQLOnly>

<PostgreSQLOnly>

- Первичный ключ.

  Если вы не определили первичный ключ с помощью параметров столбца, то это можно сделать с помощью дополнительных параметров таблицы, добавив запись `PRIMARY KEY (<столбец_1>, <столбец_n>)` после перечисления столбцов:

  ```sql
  CREATE TABLE Users (
      id INTEGER,
      name VARCHAR(255) NOT NULL,
      age INTEGER NOT NULL DEFAULT 18,
      PRIMARY KEY (id)
  );
  ```

</PostgreSQLOnly>

<MySQLOnly>

- Внешние ключи.

  Предположим, что мы хотим хранить данные о компании, в которой работают наши пользователи. Давайте создадим небольшую таблицу `Companies`, в которой мы будем хранить уникальный идентификатор и название компании:

  ```sql
  CREATE TABLE Companies (
      id INT,
      name VARCHAR(255) NOT NULL,
      PRIMARY KEY (id)
  );
  ```

  Дальше нужно добавить в таблицу `Users` поле `company` – место работы нашего пользователя, которое будет ссылаться на запись в таблице `Companies`. Полный запрос для создания таблицы будет выглядеть так:

  ```sql
  CREATE TABLE Users (
      id INT,
      name VARCHAR(255) NOT NULL,
      age INT NOT NULL DEFAULT 18,
      company INT,
      PRIMARY KEY (id)
  );
  ```

  Для того, чтобы при добавлении новых записей в таблицу `Users` гарантировать, что в колонке `company` находится идентификатор, существующий в таблице `Companies`,
  используется внешний ключ. Он имеет следующий синтаксис:

  ```sql
  FOREIGN KEY (<столбец_1>, <столбец_n>)
  REFERENCES <внешняя_таблица> (<столбец_во_внешней_таблице_1>, <столбец_во_внешней_таблице_n>)
  [ON DELETE действие]
  [ON UPDATE действие]
  ```

  Полный запрос для создания таблицы с внешним ключом будет таким:

  ```sql
  CREATE TABLE Users (
      id INT,
      name VARCHAR(255) NOT NULL,
      age INT NOT NULL DEFAULT 18,
      company INT,
      PRIMARY KEY (id),
      FOREIGN KEY (company) REFERENCES Companies (id)
  );
  ```

  При наличии внешних ключей можно определить поведение текущей записи, при изменении или удалении записи, на которую она ссылается.

  ```sql
  CREATE TABLE Users (
      id INT,
      name VARCHAR(255) NOT NULL,
      age INT NOT NULL DEFAULT 18,
      company INT,
      PRIMARY KEY (id),
      FOREIGN KEY (company) REFERENCES Companies (id)
      ON DELETE RESTRICT ON UPDATE CASCADE
  );
  ```

  `ON DELETE RESTRICT` означает, что если попробовать удалить компанию, у которой в таблице `Users` есть данные, база данных не даст этого сделать:

  ```sql
  Cannot delete or update a parent row: a foreign key constraint fails
  ```

  Если бы было указано `ON DELETE CASCADE`, то при удалении компании были бы удалены все пользователи, ссылающиеся на эту компанию.

  Есть ещё одна опция — `ON DELETE SET NULL`. При её использовании база данных запишет `NULL` в качестве значения поля `company` для всех пользователей, работавших в удалённой компании.

  `ON UPDATE CASCADE` означает, что если компания изменит свой идентификатор, то все пользователи (`Users`) получат новый идентификатор в поле company.

</MySQLOnly>

<PostgreSQLOnly>

- Внешние ключи.

  Предположим, что мы хотим хранить данные о компании, в которой работают наши пользователи. Давайте создадим небольшую таблицу `Companies`, в которой мы будем хранить уникальный идентификатор и название компании:

  ```sql
  CREATE TABLE Companies (
      id INTEGER,
      name VARCHAR(255) NOT NULL,
      PRIMARY KEY (id)
  );
  ```

  Дальше нужно добавить в таблицу `Users` поле `company` – место работы нашего пользователя, которое будет ссылаться на запись в таблице `Companies`. Полный запрос для создания таблицы будет выглядеть так:

  ```sql
  CREATE TABLE Users (
      id INTEGER,
      name VARCHAR(255) NOT NULL,
      age INTEGER NOT NULL DEFAULT 18,
      company INTEGER,
      PRIMARY KEY (id)
  );
  ```

  Для того, чтобы при добавлении новых записей в таблицу `Users` гарантировать, что в колонке `company` находится идентификатор, существующий в таблице `Companies`,
  используется внешний ключ. Он имеет следующий синтаксис:

  ```sql
  FOREIGN KEY (<столбец_1>, <столбец_n>)
  REFERENCES <внешняя_таблица> (<столбец_во_внешней_таблице_1>, <столбец_во_внешней_таблице_n>)
  [ON DELETE действие]
  [ON UPDATE действие]
  ```

  Полный запрос для создания таблицы с внешним ключом будет таким:

  ```sql
  CREATE TABLE Users (
      id INTEGER,
      name VARCHAR(255) NOT NULL,
      age INTEGER NOT NULL DEFAULT 18,
      company INTEGER,
      PRIMARY KEY (id),
      FOREIGN KEY (company) REFERENCES Companies (id)
  );
  ```

  При наличии внешних ключей можно определить поведение текущей записи, при изменении или удалении записи, на которую она ссылается.

  ```sql
  CREATE TABLE Users (
      id INTEGER,
      name VARCHAR(255) NOT NULL,
      age INTEGER NOT NULL DEFAULT 18,
      company INTEGER,
      PRIMARY KEY (id),
      FOREIGN KEY (company) REFERENCES Companies (id)
      ON DELETE RESTRICT ON UPDATE CASCADE
  );
  ```

  `ON DELETE RESTRICT` означает, что если попробовать удалить компанию, у которой в таблице `Users` есть данные, база данных не даст этого сделать:

  ```sql
  Cannot delete or update a parent row: a foreign key constraint fails
  ```

  Если бы было указано `ON DELETE CASCADE`, то при удалении компании были бы удалены все пользователи, ссылающиеся на эту компанию.

  Есть ещё одна опция — `ON DELETE SET NULL`. При её использовании база данных запишет `NULL` в качестве значения поля `company` для всех пользователей, работавших в удалённой компании.

  `ON UPDATE CASCADE` означает, что если компания изменит свой идентификатор, то все пользователи (`Users`) получат новый идентификатор в поле company.

</PostgreSQLOnly>

## Удаление таблицы

Удаление таблицы производится при помощи оператора `DROP TABLE`.

```sql
DROP TABLE [IF EXISTS] имя_таблицы;
```
