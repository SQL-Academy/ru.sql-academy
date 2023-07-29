# Операторы IS NULL, BETWEEN, IN

Мы уже познакомились с синтаксисом оператора `WHERE` и операторами сравнения, но помимо них в условных запросах мы можем использовать следующие полезные операторы:

- `IS NULL`
- `BETWEEN`
- `IN`

Давайте рассмотрим их применение.

## IS NULL

Оператор `IS NULL` позволяет узнать равно ли проверяемое значение `NULL`, т.е. пустое ли значение.

Для примера выведем всех преподавателей, у кого отсутствует отчество:

```sql
SELECT * FROM Teacher
WHERE middle_name IS NULL;
```

| id  | first_name | middle_name | last_name |
| --- | ---------- | ----------- | --------- |
| 10  | YUrij      | null        | Krylov    |
| 11  | Andrej     | null        | Evseev    |

Для использования отрицания, то есть, если мы хотим найти все записи, где поле не равно `NULL`, мы должны использовать следующий синтаксис:

```sql
SELECT * FROM Teacher
WHERE middle_name IS NOT NULL;
```

## BETWEEN

Оператор `BETWEEN min AND max` позволяет узнать расположено ли проверяемое значение столбца в интервале между `min` и `max`, включая сами значения `min` и `max`.
Он идентичен условию:

```sql
... WHERE field >= min AND field <= max
```

Используется данный оператор следующим образом:

```sql
SELECT * FROM Payments
WHERE unit_price BETWEEN 100 AND 500;
```

В качестве результата вернутся все записи из таблицы `Payments`, где значение поля `unit_price` будет от 100 до 500.

## IN

Оператор `IN` позволяет узнать входит ли проверяемое значение столбца в список определённых значений.

```sql
SELECT * FROM FamilyMembers
WHERE status IN ('father', 'mother');
```

| member_id | status | member_name       | birthday                 |
| --------- | ------ | ----------------- | ------------------------ |
| 1         | father | Headley Quincey   | 1960-05-13T00:00:00.000Z |
| 2         | mother | Flavia Quincey    | 1963-02-16T00:00:00.000Z |
| 6         | father | Ernest Forrest    | 1961-09-11T00:00:00.000Z |
| 7         | mother | Constance Forrest | 1968-09-06T00:00:00.000Z |
