# Обобщённое табличное выражение, оператор WITH

Обобщённое табличное выражение или CTE (Common Table Expressions) - это временный результирующий набор данных, к которому можно обращаться в последующих запросах.
Для написания обобщённого табличного выражения используется оператор `WITH`.

```sql
-- Пример использования конструкции WITH
WITH Aeroflot_trips AS
    (SELECT TRIP.* FROM Company
        INNER JOIN Trip ON Trip.company = Company.id WHERE name = "Aeroflot")

SELECT plane, COUNT(plane) AS amount FROM Aeroflot_trips GROUP BY plane;
```

Выражение с `WITH` считается «временным», потому что результат не сохраняется где-либо на постоянной основе
в схеме базы данных, а действует как временное представление, которое существует только на время выполнения запроса,
то есть оно доступно только во время выполнения операторов `SELECT`, `INSERT`, `UPDATE`, `DELETE` или `MERGE`.
Оно действительно только в том запросе, которому он принадлежит, что позволяет улучшить структуру запроса, не загрязняя глобальное пространство имён.

## Синтаксис оператора WITH

```sql
WITH название_cte [(столбец_1 [, столбец_2 ] …)] AS (подзапрос)
    [, название_cte [(столбец_1 [, столбец_2 ] …)] AS (подзапрос)] …
```

Порядок использования оператора `WITH`:

1. Ввести оператор `WITH`
2. Указать название обобщённого табличного выражения
3. Опционально: определить названия для столбцов получившегося табличного выражения, разделённых знаком запятой
4. Ввести `AS` и далее подзапрос, результат которого можно будет использовать в других частях SQL запроса, используя имя, определённое на 2 этапе
5. Опционально: если необходимо более одного табличного выражения, то ставится запятая и повторяются шаги 2-4

## Примеры запросов

<ERD databaseName="Airo" />

1. Создаём табличное выражение `Aeroflot_trips`, содержащие все полёты, совершенные авиакомпанией «Aeroflot»

```sql
WITH Aeroflot_trips AS
    (SELECT plane, town_from, town_to FROM Company
        INNER JOIN Trip ON Trip.company = Company.id WHERE name = "Aeroflot")

SELECT * FROM Aeroflot_trips;
```

| plane | town_from | town_to |
| ----- | --------- | ------- |
| IL-86 | Moscow    | Rostov  |
| IL-86 | Rostov    | Moscow  |

2. Аналогично, создаём табличное выражение `Aeroflot_trips`, но с переименованными колонками

```sql
WITH Aeroflot_trips (aeroflot_plane, town_from, town_to) AS
    (SELECT plane, town_from, town_to FROM Company
        INNER JOIN Trip ON Trip.company = Company.id WHERE name = "Aeroflot")

SELECT * FROM Aeroflot_trips;
```

| aeroflot_plane | town_from | town_to |
| -------------- | --------- | ------- |
| IL-86          | Moscow    | Rostov  |
| IL-86          | Rostov    | Moscow  |

3. С помощью оператора `WITH` определяем несколько табличных выражений

```sql
WITH Aeroflot_trips AS
    (SELECT TRIP.* FROM Company
        INNER JOIN Trip ON Trip.company = Company.id WHERE name = "Aeroflot"),
    Don_avia_trips AS
    (SELECT TRIP.* FROM Company
        INNER JOIN Trip ON Trip.company = Company.id WHERE name = "Don_avia")

SELECT * FROM Don_avia_trips UNION SELECT * FROM  Aeroflot_trips;
```

| id   | company | plane  | town_from | town_to | time_out                 | time_in                  |
| ---- | ------- | ------ | --------- | ------- | ------------------------ | ------------------------ |
| 1181 | 1       | TU-134 | Rostov    | Moscow  | 1900-01-01T06:12:00.000Z | 1900-01-01T08:01:00.000Z |
| 1182 | 1       | TU-134 | Moscow    | Rostov  | 1900-01-01T12:35:00.000Z | 1900-01-01T14:30:00.000Z |
| 1187 | 1       | TU-134 | Rostov    | Moscow  | 1900-01-01T15:42:00.000Z | 1900-01-01T17:39:00.000Z |
| 1188 | 1       | TU-134 | Moscow    | Rostov  | 1900-01-01T22:50:00.000Z | 1900-01-02T00:48:00.000Z |
| 1195 | 1       | TU-154 | Rostov    | Moscow  | 1900-01-01T23:30:00.000Z | 1900-01-02T01:11:00.000Z |
| 1196 | 1       | TU-154 | Moscow    | Rostov  | 1900-01-01T04:00:00.000Z | 1900-01-01T05:45:00.000Z |
| 1145 | 2       | IL-86  | Moscow    | Rostov  | 1900-01-01T09:35:00.000Z | 1900-01-01T11:23:00.000Z |
| 1146 | 2       | IL-86  | Rostov    | Moscow  | 1900-01-01T17:55:00.000Z | 1900-01-01T20:01:00.000Z |

## Заключение

Обобщённые табличные выражения были добавлены в SQL для упрощения сложных длинных запросов, особенно с множественными подзапросами. Их главная задача – улучшение читабельности,
простоты написания запросов и их дальнейшей поддержки. Это происходит за счёт сокрытия больших и сложных запросов в созданные именованные выражения, которые потом используются в основном запросе.
