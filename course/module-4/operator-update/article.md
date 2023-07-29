# Обновление данных, оператор UPDATE

Для редактирования существующих записей в таблицах существует SQL оператор `UPDATE`.

## Общая структура запроса с оператором UPDATE

```sql
UPDATE имя_таблицы
SET поле_таблицы1 = значение_поля_таблицы1,
    поле_таблицыN = значение_поля_таблицыN
[WHERE условие_выборки]
```

Так, например, если нужно изменить имя, то запрос будет иметь следующий вид:

<ERD databaseName="Family" />

```sql
UPDATE FamilyMembers
SET member_name = "Andie Anthony"
WHERE member_name = "Andie Quincey";

-- и сразу же проверим результат
SELECT * FROM FamilyMembers;
```

| member_id | status      | member_name       | birthday             |
| --------- | ----------- | ----------------- | -------------------- |
| 1         | father      | Headley Quincey   | 1960-05-13T00:00:00Z |
| 2         | mother      | Flavia Quincey    | 1963-02-16T00:00:00Z |
| 3         | varchar(50) | Andie Anthony     | 1983-06-05T00:00:00Z |
| 4         | daughter    | Lela Quincey      | 1985-06-07T00:00:00Z |
| 5         | daughter    | Annie Quincey     | 1988-04-10T00:00:00Z |
| 6         | father      | Ernest Forrest    | 1961-09-11T00:00:00Z |
| 7         | mother      | Constance Forrest | 1968-09-06T00:00:00Z |
| 8         | daughter    | Wednesday Addams  | 2005-01-13T00:00:00Z |

> Будьте внимательны, когда обновляете данные. Если вы пропустите оператор `WHERE`, то будут обновлены все записи в таблице.

## Вычисляемые значения

В запросах на обновление данных можно менять значения, опираясь на предыдущие значение.

```sql
UPDATE Payments
SET unit_price = unit_price * 2
```

Разрешается также значения одних столбцов присваивать другим столбцам. Но при этом, естественно, типы столбцов должны быть совместимыми.
