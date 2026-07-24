---
meta:
    title: "Ограничения в SQL: первичный и внешний ключ, UNIQUE, NOT NULL, CHECK"
    description: "Что такое первичный и внешний ключ и как ограничения (constraints) защищают данные: PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK и DEFAULT с примерами для MySQL и PostgreSQL."
---

# Ограничения столбцов (Constraints) в SQL

Ограничения (Constraints) — это правила, применяемые к данным в таблице для поддержания их точности и надёжности. Они играют важную роль в обеспечении целостности данных и соответствия бизнес-правилам. 🔒

Когда вы создаёте таблицу или изменяете её структуру, вы можете определить различные ограничения, которые предотвращают добавление, изменение или удаление данных, нарушающих установленные правила.
Это помогает избежать нежелательных ситуаций, таких как:

- Наличие нескольких пользователей с одинаковыми идентификаторами
- Ссылки на несуществующие записи в других таблицах
- Отсутствие обязательных данных
- Ввод некорректных значений (например, отрицательного возраста или будущей даты рождения)

## Основные типы ограничений в SQL ✨

В SQL существуют следующие основные типы ограничений:

1. **PRIMARY KEY** — уникальный идентификатор записи в таблице
2. **FOREIGN KEY** — обеспечивает ссылочную целостность между таблицами
3. **UNIQUE** — гарантирует уникальность значений в столбце или группе столбцов
4. **NOT NULL** — запрещает NULL-значения в столбце
5. **CHECK** — проверяет соответствие данных заданному условию
6. **DEFAULT** — устанавливает значение по умолчанию для столбца

Давайте рассмотрим каждый тип подробнее.

## PRIMARY KEY (Первичный ключ)

Первичный ключ — это столбец или комбинация столбцов, которые однозначно идентифицируют каждую строку в таблице.

Он не может содержать NULL-значения и должен быть уникальным. Таблица может иметь только один первичный ключ.

**MySQL**

```sql
CREATE TABLE Users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);
```

Альтернативный синтаксис с использованием отдельного ограничения:

```sql
CREATE TABLE Users (
    id INT AUTO_INCREMENT,
    username VARCHAR(50),
    email VARCHAR(100),
    CONSTRAINT pk_users PRIMARY KEY (id)
);
```

При попытке добавить запись с уже существующим первичным ключом или с NULL-значением вместо ключа СУБД выдаст ошибку:

```sql
Error(1062) 23000: "Duplicate entry '1' for key 'users.PRIMARY'"
```

**PostgreSQL**

```sql
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100)
);
```

Альтернативный синтаксис с использованием отдельного ограничения:

```sql
CREATE TABLE Users (
    id SERIAL,
    username VARCHAR(50),
    email VARCHAR(100),
    CONSTRAINT pk_users PRIMARY KEY (id)
);
```

При попытке добавить запись с уже существующим первичным ключом или с NULL-значением вместо ключа, СУБД выдаст ошибку:

```sql
ERROR: duplicate key value violates unique constraint "users_pkey"
DETAIL: Key (id)=(1) already exists.
```

## FOREIGN KEY (Внешний ключ)

Внешний ключ — это столбец или группа столбцов в одной таблице, которые ссылаются на первичный ключ другой таблицы.

Он обеспечивает ссылочную целостность данных, гарантируя, что значения в столбце внешнего ключа соответствуют значениям из столбца первичного ключа связанной таблицы.

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES Users(id)
);
```

Благодаря ограничению FOREIGN KEY:

- Нельзя добавить заказ для несуществующего пользователя
- Нельзя удалить пользователя, у которого есть заказы (если не указаны специальные опции CASCADE)

Можно также указать действия, которые должны быть выполнены при удалении или обновлении связанных данных:

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES Users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

Возможные опции для `ON DELETE` и `ON UPDATE`:

- `CASCADE` — автоматически удаляет или обновляет связанные записи
- `SET NULL` — устанавливает NULL для внешнего ключа
- `SET DEFAULT` — устанавливает значение по умолчанию
- `RESTRICT` — запрещает удаление или обновление (используется по умолчанию)
- `NO ACTION` — аналогично RESTRICT в большинстве СУБД

## UNIQUE (Уникальность)

Ограничение UNIQUE гарантирует, что все значения в столбце или группе столбцов уникальны. В отличие от PRIMARY KEY, оно допускает NULL-значения (обычно только одно NULL-значение, так как NULL != NULL).

```sql
CREATE TABLE Users (
    id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE
);
```

Это предотвратит создание нескольких пользователей с одинаковым именем пользователя или адресом электронной почты.

## NOT NULL (Запрет пустых значений)

Ограничение NOT NULL гарантирует, что столбец не может содержать NULL-значения. Это полезно для обязательных полей, без которых запись не имеет смысла.

```sql
CREATE TABLE Users (
    id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    bio TEXT
);
```

В этом примере поля `username` и `email` обязательны, а `bio` может быть пустым.

## CHECK (Проверка условия)

**MySQL**

Ограничение CHECK позволяет определить условие, которому должны соответствовать значения в столбце. Это помогает обеспечить бизнес-правила и предотвратить ввод некорректных данных.

**Примечание:** CHECK ограничения полностью поддерживаются в MySQL, начиная с версии 8.0.16. В более ранних версиях они принимались синтаксически, но не проверялись.

```sql
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price > 0),
    quantity INT CHECK (quantity >= 0)
);
```

Более сложный пример с именованным ограничением:

```sql
CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    birth_date DATE NOT NULL,
    hire_date DATE NOT NULL,
    CONSTRAINT chk_dates CHECK (hire_date > birth_date)
);
```

**PostgreSQL**

Ограничение CHECK позволяет определить условие, которому должны соответствовать значения в столбце. Это помогает обеспечить бизнес-правила и предотвратить ввод некорректных данных.

PostgreSQL полностью поддерживает CHECK ограничения и активно их проверяет.

```sql
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price > 0),
    quantity INT CHECK (quantity >= 0)
);
```

Более сложный пример с именованным ограничением:

```sql
CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    birth_date DATE NOT NULL,
    hire_date DATE NOT NULL,
    CONSTRAINT chk_dates CHECK (hire_date > birth_date)
);
```

PostgreSQL также поддерживает сложные CHECK ограничения с подзапросами и функциями:

```sql
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    age INT CHECK (age >= 0 AND age <= 150)
);
```

## DEFAULT (Значение по умолчанию)

Ограничение DEFAULT устанавливает значение, которое будет использовано, если при добавлении новой записи не указано значение для этого столбца.

**MySQL**

```sql
CREATE TABLE Orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    order_date DATE DEFAULT (CURRENT_DATE),
    status VARCHAR(20) DEFAULT 'Pending',
    FOREIGN KEY (user_id) REFERENCES Users(id)
);
```

В этом примере, если не указана дата заказа, будет использована текущая дата, а статус по умолчанию будет установлен как "Pending".

**PostgreSQL**

```sql
CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT,
    order_date DATE DEFAULT CURRENT_DATE,
    status VARCHAR(20) DEFAULT 'Pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(id)
);
```

В этом примере, если не указана дата заказа, будет использована текущая дата, а статус по умолчанию будет установлен как "Pending".

PostgreSQL также поддерживает более сложные DEFAULT значения, включая вызовы функций:

```sql
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW(),
    uuid_field UUID DEFAULT gen_random_uuid()
);
```

## Добавление и удаление ограничений

**MySQL**

Ограничения можно добавлять не только при создании таблицы, но и при её изменении:

```sql
-- Добавление ограничения PRIMARY KEY
ALTER TABLE Users
ADD PRIMARY KEY (id);

-- Добавление ограничения FOREIGN KEY
ALTER TABLE Orders
ADD CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES Users(id);

-- Добавление ограничения UNIQUE
ALTER TABLE Users
ADD CONSTRAINT uq_email UNIQUE (email);

-- Добавление ограничения CHECK
ALTER TABLE Products
ADD CONSTRAINT chk_price CHECK (price > 0);

-- Добавление ограничения NOT NULL
ALTER TABLE Users
MODIFY username VARCHAR(50) NOT NULL;

-- Добавление значения по умолчанию
ALTER TABLE Orders
ALTER COLUMN status SET DEFAULT 'Pending';
```

Удаление ограничений также выполняется с помощью команды ALTER TABLE:

```sql
-- Удаление ограничения PRIMARY KEY
ALTER TABLE Users
DROP PRIMARY KEY;

-- Удаление ограничения FOREIGN KEY
ALTER TABLE Orders
DROP FOREIGN KEY fk_user;

-- Удаление ограничения UNIQUE
ALTER TABLE Users
DROP INDEX uq_email;

-- Удаление ограничения CHECK
ALTER TABLE Products
DROP CHECK chk_price;

-- Удаление ограничения NOT NULL
ALTER TABLE Users
MODIFY username VARCHAR(50) NULL;

-- Удаление значения по умолчанию
ALTER TABLE Orders
ALTER COLUMN status DROP DEFAULT;
```

**PostgreSQL**

Ограничения можно добавлять не только при создании таблицы, но и при её изменении:

```sql
-- Добавление ограничения PRIMARY KEY
ALTER TABLE Users
ADD PRIMARY KEY (id);

-- Добавление ограничения FOREIGN KEY
ALTER TABLE Orders
ADD CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES Users(id);

-- Добавление ограничения UNIQUE
ALTER TABLE Users
ADD CONSTRAINT uq_email UNIQUE (email);

-- Добавление ограничения CHECK
ALTER TABLE Products
ADD CONSTRAINT chk_price CHECK (price > 0);

-- Добавление ограничения NOT NULL
ALTER TABLE Users
ALTER COLUMN username SET NOT NULL;

-- Добавление значения по умолчанию
ALTER TABLE Orders
ALTER COLUMN status SET DEFAULT 'Pending';
```

Удаление ограничений также выполняется с помощью команды ALTER TABLE:

```sql
-- Удаление ограничения PRIMARY KEY
ALTER TABLE Users
DROP CONSTRAINT users_pkey;

-- Удаление ограничения FOREIGN KEY
ALTER TABLE Orders
DROP CONSTRAINT fk_user;

-- Удаление ограничения UNIQUE
ALTER TABLE Users
DROP CONSTRAINT uq_email;

-- Удаление ограничения CHECK
ALTER TABLE Products
DROP CONSTRAINT chk_price;

-- Удаление ограничения NOT NULL
ALTER TABLE Users
ALTER COLUMN username DROP NOT NULL;

-- Удаление значения по умолчанию
ALTER TABLE Orders
ALTER COLUMN status DROP DEFAULT;
```

## Лучшие практики использования ограничений 🚀

При проектировании базы данных следует придерживаться следующих рекомендаций:

1. **Всегда определяйте первичный ключ** для каждой таблицы, чтобы однозначно идентифицировать каждую запись.

2. **Используйте внешние ключи** для обеспечения ссылочной целостности между связанными таблицами.

3. **Применяйте ограничение NOT NULL** для столбцов, которые должны содержать значения.

4. **Используйте ограничение UNIQUE** для столбцов, которые должны содержать уникальные значения (например, email, номер телефона).

5. **Добавляйте ограничения CHECK** для столбцов, значения которых должны соответствовать определенным бизнес-правилам.

6. **Устанавливайте значения по умолчанию** для столбцов, которые часто принимают одно и то же значение.

7. **Указывайте имена для ограничений** (например, `CONSTRAINT pk_users PRIMARY KEY (id)`), чтобы облегчить их идентификацию и управление.

## Проверка знаний об ограничениях в SQL:

Какое из следующих ограничений в SQL НЕ может содержать NULL-значения?

1. UNIQUE — Ограничение UNIQUE допускает NULL-значения, хотя обычно только одно NULL-значение в столбце.

2. CHECK — Ограничение CHECK проверяет условие только для непустых значений, NULL-значения обычно пропускаются проверкой.

3. **Правильный ответ:&#x20;**&#x50;RIMARY KEY — Первичный ключ не может содержать NULL-значения, так как он должен уникально идентифицировать каждую строку в таблице.

4. FOREIGN KEY — Внешний ключ может содержать NULL-значения, если не указано иное, это означает отсутствие связи с другой таблицей.
