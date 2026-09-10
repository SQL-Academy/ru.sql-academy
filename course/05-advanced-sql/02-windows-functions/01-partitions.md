---
meta:
    title: "Партиции в оконных функциях"
    description: "Партиции в оконных функциях SQL. Использование партиций по нескольким колонкам. Синтаксис партиций."
---

# Партиции в оконных функциях

В <a href="https://sql-academy.org/ru/guide/windows-functions" target="_blank">прошлой статье</a> мы кратко уже упоминали, что
такое партиции и как их использовать в оконных функциях, пришло время разобраться в них поподробнее 🤓.

## Понятие партиции

> Партиции — подмножества строк, выделенные для оконной функции на основе одного или нескольких столбцов в таблице.

Они служат для сегментации данных, позволяя выполнить более детальный анализ и
расчёты вроде агрегации или ранжирования внутри каждой такой группы.

Применяя партиционирование, например, по типу жилья в таблице с данными о цене жилья,
мы можем рассчитать в отдельной колонке, скажем, среднюю цену для каждого типа жилья.

![Схема разбиения на партиции](https://sql-academy.org/static/guidePage/windows-functions/3.png "Схема разбиения на партиции")

## Применение партиций в SQL

Для того чтобы использовать партицию вместе с оконной функцией, необходимо придерживаться следующего
синтаксиса:

```sql
SELECT <оконная_функция>(<поле_таблицы>)
OVER (
    PARTITION BY <столбцы_для_разделения>
)
```

### Пример использования

А теперь давайте на простом примере рассмотрим использование партиции вместе с оконной функцией.

ER-диаграмма базы данных Airbnb: [открыть на SQL Academy](https://sql-academy.org/ru/guide/partitions).

Для этого рассмотрим таблицу `Rooms`, а именно поля `home_type` и `price`:

```sql
SELECT home_type, price FROM Rooms;
```

| home_type       | price |
| --------------- | ----- |
| Private room    | 149   |
| Entire home/apt | 225   |
| Private room    | 150   |
| Entire home/apt | 89    |
| Entire home/apt | 80    |
| Entire home/apt | 200   |
| Private room    | 60    |
| Private room    | 79    |
| Private room    | 79    |
| Entire home/apt | 150   |
| Entire home/apt | 135   |
| Private room    | 85    |
| Private room    | 89    |
| Private room    | 85    |
| Entire home/apt | 120   |
| Entire home/apt | 140   |
| Entire home/apt | 215   |
| Private room    | 140   |
| Entire home/apt | 99    |
| Entire home/apt | 190   |
| Entire home/apt | 299   |
| Private room    | 130   |
| Private room    | 80    |
| Private room    | 110   |
| Entire home/apt | 120   |
| Private room    | 60    |
| Private room    | 80    |
| Entire home/apt | 150   |
| Private room    | 44    |
| Entire home/apt | 180   |
| Private room    | 50    |
| Private room    | 52    |
| Private room    | 55    |
| Private room    | 50    |
| Private room    | 70    |
| Private room    | 89    |
| Private room    | 35    |
| Entire home/apt | 85    |
| Private room    | 150   |
| Shared room     | 40    |
| Private room    | 68    |
| Entire home/apt | 120   |
| Private room    | 120   |
| Private room    | 135   |
| Entire home/apt | 150   |
| Entire home/apt | 150   |
| Private room    | 130   |
| Entire home/apt | 110   |
| Entire home/apt | 115   |
| Private room    | 80    |

Мы можем увидеть, что все жильё для аренды разделено на 3 категории: «Private room», «Entire home/apt» и «Shared room».

Каждая категория жилья имеет свои ценовые рамки.
Чтобы узнать среднюю стоимость в конкретной категории и сравнить её с текущей, как раз можно использовать оконные функции.

Для этого добавим к нашей результирующей таблице ещё одно поле `avg_price`, которое будет высчитывать среднюю цену по категориям. Это будет выглядеть следующим образом:

```sql
SELECT
    home_type, price,
    AVG(price) OVER (PARTITION BY home_type) AS avg_price
FROM Rooms
```

| home_type       | price | avg_price |
| --------------- | ----- | --------- |
| Entire home/apt | 225   | 148.6667  |
| Entire home/apt | 180   | 148.6667  |
| Entire home/apt | 150   | 148.6667  |
| Entire home/apt | 85    | 148.6667  |
| Entire home/apt | 120   | 148.6667  |
| Entire home/apt | 120   | 148.6667  |
| Entire home/apt | 299   | 148.6667  |
| Entire home/apt | 190   | 148.6667  |
| Entire home/apt | 99    | 148.6667  |
| Entire home/apt | 215   | 148.6667  |
| Entire home/apt | 140   | 148.6667  |
| Entire home/apt | 120   | 148.6667  |
| Entire home/apt | 150   | 148.6667  |
| Entire home/apt | 135   | 148.6667  |
| Entire home/apt | 150   | 148.6667  |
| Entire home/apt | 110   | 148.6667  |
| Entire home/apt | 115   | 148.6667  |
| Entire home/apt | 200   | 148.6667  |
| Entire home/apt | 150   | 148.6667  |
| Entire home/apt | 80    | 148.6667  |
| Entire home/apt | 89    | 148.6667  |
| Private room    | 68    | 89.4286   |
| Private room    | 50    | 89.4286   |
| Private room    | 70    | 89.4286   |
| Private room    | 80    | 89.4286   |
| Private room    | 89    | 89.4286   |
| Private room    | 149   | 89.4286   |
| Private room    | 35    | 89.4286   |
| Private room    | 150   | 89.4286   |
| Private room    | 130   | 89.4286   |
| Private room    | 120   | 89.4286   |
| Private room    | 135   | 89.4286   |
| Private room    | 130   | 89.4286   |
| Private room    | 150   | 89.4286   |
| Private room    | 60    | 89.4286   |
| Private room    | 79    | 89.4286   |
| Private room    | 79    | 89.4286   |
| Private room    | 85    | 89.4286   |
| Private room    | 89    | 89.4286   |
| Private room    | 85    | 89.4286   |
| Private room    | 140   | 89.4286   |
| Private room    | 55    | 89.4286   |
| Private room    | 80    | 89.4286   |
| Private room    | 110   | 89.4286   |
| Private room    | 60    | 89.4286   |
| Private room    | 80    | 89.4286   |
| Private room    | 44    | 89.4286   |
| Private room    | 50    | 89.4286   |
| Private room    | 52    | 89.4286   |
| Shared room     | 40    | 40        |

Что именно происходит в добавленной строке?

- `PARTITION BY home_type` делит все записи на разные партиции на основе уникальных значений столбца `home_type`
- затем для каждой записи `AVG(price)` вычисляет среднюю цену (`price`) в рамках её партиции (`home_type`)

Результатом выполнения этой части запроса будет столбец `avg_price`,
в котором для каждой записи будет указано среднее значение цены для её категории жилья (`home_type`).

## Партиции по нескольким колонкам

Партиционирование также может быть выполнено по нескольким колонкам. Это позволяет создавать более сложные и точные сегменты для анализа.

Например, для нашей таблицы `Rooms` мы можем создать партиции на основании 2 колонок: категория жилья
`home_type` и наличие телевизора в жилье `has_tv`.

Пример запроса с партиционированием по двум столбцам:

```sql
SELECT
    home_type, has_tv, price,
    AVG(price) OVER (PARTITION BY home_type, has_tv) AS avg_price
    FROM Rooms
```

| home_type       | has_tv | price | avg_price |
| --------------- | ------ | ----- | --------- |
| Entire home/apt | 0      | 225   | 170       |
| Entire home/apt | 0      | 180   | 170       |
| Entire home/apt | 0      | 80    | 170       |
| Entire home/apt | 0      | 200   | 170       |
| Entire home/apt | 0      | 150   | 170       |
| Entire home/apt | 0      | 150   | 170       |
| Entire home/apt | 0      | 190   | 170       |
| Entire home/apt | 0      | 215   | 170       |
| Entire home/apt | 0      | 140   | 170       |
| Entire home/apt | 1      | 99    | 132.6667  |
| Entire home/apt | 1      | 85    | 132.6667  |
| Entire home/apt | 1      | 150   | 132.6667  |
| Entire home/apt | 1      | 120   | 132.6667  |
| Entire home/apt | 1      | 120   | 132.6667  |
| Entire home/apt | 1      | 299   | 132.6667  |
| Entire home/apt | 1      | 120   | 132.6667  |
| Entire home/apt | 1      | 135   | 132.6667  |
| Entire home/apt | 1      | 150   | 132.6667  |
| Entire home/apt | 1      | 110   | 132.6667  |
| Entire home/apt | 1      | 89    | 132.6667  |
| Entire home/apt | 1      | 115   | 132.6667  |
| Private room    | 0      | 85    | 78.5455   |
| Private room    | 0      | 35    | 78.5455   |
| Private room    | 0      | 150   | 78.5455   |
| Private room    | 0      | 55    | 78.5455   |
| Private room    | 0      | 52    | 78.5455   |
| Private room    | 0      | 50    | 78.5455   |
| Private room    | 0      | 68    | 78.5455   |
| Private room    | 0      | 60    | 78.5455   |
| Private room    | 0      | 135   | 78.5455   |
| Private room    | 0      | 85    | 78.5455   |
| Private room    | 0      | 89    | 78.5455   |
| Private room    | 1      | 120   | 96.4706   |
| Private room    | 1      | 80    | 96.4706   |
| Private room    | 1      | 149   | 96.4706   |
| Private room    | 1      | 130   | 96.4706   |
| Private room    | 1      | 89    | 96.4706   |
| Private room    | 1      | 70    | 96.4706   |
| Private room    | 1      | 50    | 96.4706   |
| Private room    | 1      | 44    | 96.4706   |
| Private room    | 1      | 80    | 96.4706   |
| Private room    | 1      | 60    | 96.4706   |
| Private room    | 1      | 110   | 96.4706   |
| Private room    | 1      | 80    | 96.4706   |
| Private room    | 1      | 130   | 96.4706   |
| Private room    | 1      | 140   | 96.4706   |
| Private room    | 1      | 79    | 96.4706   |
| Private room    | 1      | 79    | 96.4706   |
| Private room    | 1      | 150   | 96.4706   |
| Shared room     | 1      | 40    | 40        |

Здесь `PARTITION BY home_type, has_tv` создаёт уникальные партиции для каждой комбинации `home_type` и `has_tv`,
позволяя вычислить среднюю цену жилья для текущей категории жилья с наличием или без наличия телевизора.

![Партиции по 2 колонками](https://sql-academy.org/static/guidePage/partitions/2-columns-partition.png "Партиции по 2 колонками")
