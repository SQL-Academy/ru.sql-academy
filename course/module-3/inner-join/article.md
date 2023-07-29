# Внутреннее соединение INNER JOIN

В предыдущем уроке мы рассмотрели общую структуру многотабличного запроса:

```sql
SELECT поля_таблиц
FROM таблица_1
[INNER] | [[LEFT | RIGHT | FULL][OUTER]] JOIN таблица_2
    ON условие_соединения
[INNER] | [[LEFT | RIGHT | FULL][OUTER]] JOIN таблица_n
    ON условие_соединения]
```

Говоря о многотабличном запросе со внутренним соединением общая структура выглядит так:

```sql
SELECT поля_таблиц
FROM таблица_1
[INNER] JOIN таблица_2
    ON условие_соединения
[INNER] JOIN таблица_n
    ON условие_соединения]
```

Например, запрос может выглядеть следующим образом:

<ERD databaseName="Family" />

```sql
SELECT family_member, member_name FROM Payments
INNER JOIN FamilyMembers
    ON Payments.family_member = FamilyMembers.member_id
```

| family_member | member_name     |
| ------------- | --------------- |
| 1             | Headley Quincey |
| 2             | Flavia Quincey  |
| 3             | Andie Quincey   |
| 4             | Lela Quincey    |
| 4             | Lela Quincey    |
| 5             | Annie Quincey   |
| 2             | Flavia Quincey  |
| 2             | Flavia Quincey  |
| 5             | Annie Quincey   |
| 3             | Andie Quincey   |
| 2             | Flavia Quincey  |
| 1             | Headley Quincey |
| 3             | Andie Quincey   |
| 3             | Andie Quincey   |

Так как, по умолчанию, если не указаны какие-либо параметры, `JOIN` выполняется как `INNER JOIN`, то при внутреннем соединении `INNER` является опциональным.

## Понятие внутреннего соединения

Внутреннее соединение — соединение, при котором находятся пары записей из двух таблиц, удовлетворяющие условию соединения, тем самым образуя новую таблицу, содержащую
поля из первой и второй исходных таблиц.

Для наглядности это выглядит следующим образом:

![Схема внутреннего соединения](https://sql-academy.org/static/guidePage/inner-join/inner-join-example.png "Схема внутреннего соединения")

Так как в нашем условии указано равенство полей `Payments.good_id` и `Goods.good_id`, то при внутреннем соединении в итоговой выборке окажутся только записи,
где в обоих таблица есть одинаковое значение `good_id`.

## Использование WHERE для соединения таблиц

Для внутреннего соединения таблиц также можно использовать оператор `WHERE`. Например, вышеприведённый запрос, написанный с помощью `INNER JOIN`, будет выглядеть так:

```sql
SELECT family_member, member_name FROM Payments, FamilyMembers
    WHERE Payments.family_member = FamilyMembers.member_id
```

| family_member | member_name     |
| ------------- | --------------- |
| 1             | Headley Quincey |
| 2             | Flavia Quincey  |
| 3             | Andie Quincey   |
| 4             | Lela Quincey    |
| 4             | Lela Quincey    |
| 5             | Annie Quincey   |
| 2             | Flavia Quincey  |
| 2             | Flavia Quincey  |
| 5             | Annie Quincey   |
| 3             | Andie Quincey   |
| 2             | Flavia Quincey  |
| 1             | Headley Quincey |
| 3             | Andie Quincey   |
| 3             | Andie Quincey   |
