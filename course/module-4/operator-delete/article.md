---
meta:
    title: 'Удаление данных, оператор DELETE'
    description: 'Удалить записи в sql. SQL операторы delete, truncate и их отличия. Delete запрос c join'
---

# Удаление данных, оператор DELETE

Время от времени возникает задача удаления записей из таблицы. Для этого в SQL предусмотрены операторы `DELETE` и `TRUNCATE`,
из которых наиболее универсальным и безопасным является первый вариант.

## Общая структура запроса с оператором DELETE

```sql
DELETE FROM имя_таблицы
[WHERE условие_отбора_записей];
```

Если условие отбора записей `WHERE` отсутствует, то будут удалены все записи указанной таблицы.

Эту же операцию (удаления всех записей) можно сделать также с помощью оператора `TRUNCATE`.
Он выполнит удаление таблицы и пересоздаст её заново - этот вариант работает гораздо быстрее, чем удаление всех записей одна за другой (как в случае с `DELETE`) особенно для больших таблиц.

## Общая структура запроса с оператором TRUNCATE

```sql
TRUNCATE TABLE имя_таблицы;
```

<MySQLOnly>

> Оптимизатор запросов СУБД MySQL автоматически использует оператор `TRUNCATE`, если оператор `DELETE` не содержит условия `WHERE` или конструкции `LIMIT`.

</MySQLOnly>

Однако у оператора `TRUNCATE` есть ряд отличий:

-   Не срабатывают триггеры, в частности, триггер удаления
-   Удаляет все строки в таблице, не записывая при этом удаление отдельных строк данных в журнал транзакций
-   Сбрасывает счётчик идентификаторов до начального значения
-   Чтобы использовать, необходимы права на изменение таблицы

## Удаление записей при многотабличных запросах

Если в `DELETE` запросе используется `JOIN`, то необходимо указать из каких(ой) именно таблиц(ы) требуется удалять записи.

<MySQLOnly>

```sql
DELETE имя_таблицы_1 FROM
имя_таблицы_1 JOIN имя_таблицы_2
ON имя_таблицы_1.поле = имя_таблицы_2.поле
[WHERE условие_отбора_записей];
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
DELETE FROM имя_таблицы_1
USING имя_таблицы_2
WHERE имя_таблицы_1.поле = имя_таблицы_2.поле
[AND условие_отбора_записей];
```

</PostgreSQLOnly>

Например, нам необходимо удалить все бронирования жилья, в котором отсутствует кухня. Тогда запрос будет выглядеть следующим образом:

<MySQLOnly>

```sql
DELETE Reservations FROM
Reservations JOIN Rooms ON
Reservations.room_id = Rooms.id
WHERE Rooms.has_kitchen = false;
```

</MySQLOnly>

<PostgreSQLOnly>

```sql
DELETE FROM Reservations
USING Rooms
WHERE Reservations.room_id = Rooms.id
AND Rooms.has_kitchen = false;
```

</PostgreSQLOnly>

Если бы, помимо удаления бронирования, нам нужно было также удалить и жилье, то запрос приобрёл бы следующий вид:

<MySQLOnly>

```sql
DELETE Reservations, Rooms FROM
Reservations JOIN Rooms ON
Reservations.room_id = Rooms.id
WHERE Rooms.has_kitchen = false;
```

</MySQLOnly>

<PostgreSQLOnly>

В PostgreSQL для удаления из нескольких таблиц одновременно используются отдельные DELETE запросы или транзакции:

```sql
BEGIN;
DELETE FROM Reservations
USING Rooms
WHERE Reservations.room_id = Rooms.id
AND Rooms.has_kitchen = false;

DELETE FROM Rooms
WHERE Rooms.has_kitchen = false;
COMMIT;
```

</PostgreSQLOnly>
