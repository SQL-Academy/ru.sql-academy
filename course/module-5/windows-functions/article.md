---
meta:
    title: "Оконные функции SQL: MySQL и PostgreSQL"
    description: "Оконные функции SQL в MySQL и PostgreSQL, синтаксис OVER окна данных, оконной функции, пример использования оконной функции, очередь выполнения оконных функций в SELECT запросе"
---

![Иллюстрация к статье](https://sql-academy.org/static/guidePage/windows-functions/banner.jpg)# Оконные функции SQLОконные функции — мощный инструмент языка SQL, позволяющий проводить сложные вычисления по группам строк,
которые связаны с текущей строкой.## Принцип работыВозможно, вы зададитесь вопросом: «Что значит оконные?».В стандартном SQL-запросе все наборы строк рассматриваются как один сплошной блок данных,
для которого и вычисляются агрегатные значения.Однако, когда применяются оконные функции, запрос сегментируется на группы строк (или «окна»),
и для каждого такого сегмента подсчитываются индивидуальные агрегатные значения.Это окно, которое подаётся в оконную функцию, может быть:\* всей таблицей

- отдельными партициями таблицы, то есть группой строк на основе одного или нескольких полей
- или даже конкретным диапазоном строк в пределах таблицы или партиции.
  Например, мы можем определить окно, которое будет передаваться в оконную функцию,
  как предыдущая + текущая строка таблицы. И тогда для каждой строки значение агрегатной функции будет
  подсчитываться по-своему, так как данные, которые поступают в функцию, будут динамически меняться
  от строки к строке. Окно будет как бы «скользить» по таблице.### ВизуализацияОконные функции всегда принимают на вход окно данных, которое указывает пользователь, и возвращают результат в отдельный столбец.Давайте рассмотрим как это может выглядеть. Для этого возьмём оконную функцию `AVG` для вычисления среднего значения и вот
  такую небольшую таблицу:![Изначальная таблица](https://sql-academy.org/static/guidePage/windows-functions/schema_table.png "Изначальная таблица")А теперь давайте посмотрим, как оконная функция будет работать для разных переданных окон:\* Если в качестве окна указать всю таблицу, то для всех строк окно будет совпадать, и на вход функции `AVG` будет
  поступать один и тот же набор данных, и, соответственно, результат будет одинаковый.

    ![Схема разбиения на партиции](https://sql-academy.org/static/guidePage/windows-functions/2.png "Схема разбиения на партиции")

- Если в качестве окна указать партицию по полю `home_type`, то на вход функции `AVG` будет
  поступать набор жилых помещений с одинаковым типом, и, соответственно, в результате в новой колонке будет
  отображаться средняя стоимость по жилью, чей тип совпадает с типом у текущей строки таблицы.

    ![Схема разбиения на партиции](https://sql-academy.org/static/guidePage/windows-functions/3.png "Схема разбиения на партиции")

- В качестве окна можно указать и более специфический набор строк. Например, окно можно определить как "предыдущая + текущая строка"
  таблицы. Тогда это будет выглядеть следующим образом:

    ![Схема разбиения на партиции](https://sql-academy.org/static/guidePage/windows-functions/4.png "Схема разбиения на партиции")

    Стоит отметить, что для первой строки окно будет состоять только из 1-ой записи, так как предыдущей строки нет.## Синтаксис оконной функции`sql
SELECT <оконная_функция>(<поле_таблицы>)
OVER (
      [PARTITION BY <столбцы_для_разделения>]
      [ORDER BY <столбцы_для_сортировки>]
      [ROWS|RANGE <определение_диапазона_строк>]
)
`Где:\* `<оконная_функция>(<поле_таблицы>)` — используемая оконная функция. Например `AVG(price)`.

- Далее следует `OVER`, который определяет окно (группу строк), которое будет передаваться в оконную функцию.
  Если конструкцию `OVER ()` оставить без параметров, то окном будет выступать вся таблица.Далее внутри `OVER` следуют 3 необязательных параметра, с помощью которых можно гибко настраивать окно:\* с помощью `PARTITION BY <столбцы_для_разделения>` выборка делится на
  непересекающиеся подмножества, где каждое подмножество содержит строки с одинаковыми значениями в одном или нескольких столбцах, образуются партиции.
- с помощью `ORDER BY <столбцы_для_сортировки>` устанавливается порядок строк внутри окна, особо важную роль играет в оконных функциях ранжирования.
- с помощью `ROWS|RANGE <определение_диапазона_строк>` формируются диапазоны строк. С помощью этого параметра можно указать, сколько строк брать до и после
  текущей в окно.На каждом из этих параметров мы подробнее остановимся в следующих статьях.## Пример использования оконной функцииДавайте с помощью оконных функций попробуем получить список имён студентов и то, сколько человек у них в классе.ER-диаграмма базы данных Schedule: [открыть на SQL Academy](https://sql-academy.org/ru/guide/windows-functions).Для начала давайте получим список студентов и идентификатор класса, в котором они учатся:```sql
  SELECT
  Student.first_name,
  Student.last_name,
  Student_in_class.class
  FROM
  Student_in_class
  JOIN
  Student ON Student_in_class.student = Student.id;

````| first_name | last_name   | class |
| ----------- | ------------ | ----- |
| Nikolaj     | Sokolov      | 9     |
| Vyacheslav  | Eliseev      | 9     |
| Ivan        | Efremov      | 9     |
| Anatolij    | ZHdanov      | 9     |
| Georgij     | Noskov       | 9     |
| Artyom      | Sergeev      | 9     |
| Arina       | Evseeva      | 9     |
| Angelina    | Voroncova    | 9     |
| Ekaterina   | Ustinova     | 9     |
| Raisa       | Lapina       | 9     |
| Leonid      | Ignatov      | 9     |
| Snezhana    | Seliverstova | 9     |
| Semyon      | Biryukov     | 9     |
| Georgij     | Baranov      | 8     |
| YUliya      | Vishnyakova  | 8     |
| Valentina   | Bolshakova   | 8     |
| Leonid      | Kryukov      | 8     |
| Vladislav   | Cvetkov      | 8     |
| Snezhana    | Morozova     | 8     |
| Lyubov      | Borisova     | 8     |
| Anfisa      | Kalashnikova | 8     |
| Anna        | Osipova      | 8     |
| Kristina    | Myasnikova   | 8     |
| Kristina    | Smirnova     | 8     |
| Boris       | Simonov      | 7     |
| Dmitrij     | Trofimov     | 7     |
| YAkov       | Rozhkov      | 7     |
| Fyodor      | Drozdov      | 7     |
| Gleb        | Strelkov     | 7     |
| Angelina    | Lukina       | 7     |
| Nina        | Odincova     | 7     |
| Valeriya    | Novikova     | 7     |
| Grigorij    | Kapustin     | 7     |
| Vitalij     | Panfilov     | 7     |
| Svyatoslav  | Tarasov      | 6     |
| Matvej      | YAkushev     | 6     |
| Ilya        | Alekseev     | 6     |
| Lyubov      | Zaharova     | 6     |
| Polina      | Sidorova     | 6     |
| Elizaveta   | Samojlova    | 6     |
| YUliya      | Avdeeva      | 6     |
| Matvej      | Bogdanov     | 6     |
| Ilya        | Filippov     | 6     |
| Denis       | Mel          | 6     |
| Svyatoslav  | Muravyov     | 6     |
| Anna        | Kulagina     | 5     |
| ZHanna      | Fokina       | 5     |
| Valeriya    | Lapina       | 5     |
| Valentina   | Sazonova     | 5     |
| Nataliya    | Myasnikova   | 5     |
| Viktoriya   | Makarova     | 5     |
| Stanislav   | Lazarev      | 5     |
| Gennadij    | Ovchinnikov  | 5     |
| Roman       | SHilov       | 4     |
| Timur       | Subbotin     | 4     |
| Danila      | Osipov       | 4     |
| Arina       | Silina       | 4     |
| Nadezhda    | Zaharova     | 4     |
| Larisa      | SHCHerbakova | 4     |
| Aleksandra  | Belozyorova  | 4     |
| Natalya     | Davydova     | 4     |
| Mariya      | Fadeeva      | 4     |
| YUrij       | Markov       | 3     |
| Kirill      | SHubin       | 3     |
| Grigorij    | Kolobov      | 3     |
| Semyon      | Trofimov     | 3     |
| Vasilij     | Ustinov      | 3     |
| Valentina   | SHarova      | 3     |
| Larisa      | Savina       | 3     |
| Galina      | Orekhova     | 3     |
| Arina       | SHarapova    | 2     |
| Viktoriya   | Sergeeva     | 2     |
| Vasilij     | Krasilnikov  | 2     |
| Timur       | Rusakov      | 2     |
| Gleb        | Nesterov     | 2     |
| Denis       | Makarov      | 2     |
| Elizaveta   | SHilova      | 2     |
| Vera        | Evseeva      | 1     |
| Margarita   | Kabanova     | 1     |
| Angelina    | Lazareva     | 1     |
| Semyon      | Voronov      | 1     |
| Innokentij  | Nekrasov     | 1     |
| Artyom      | Nikitin      | 1     |
| Egor        | Belyakov     | 1     |А теперь, чтобы вычислить, сколько учащихся учится в каждом из классов и вывести эту информацию в новую колонку,
мы можем применить оконную функцию:```sql
SELECT
    Student.first_name,
    Student.last_name,
    Student_in_class.class,
    COUNT(*) OVER (PARTITION BY Student_in_class.class) AS student_count_in_class
FROM
    Student_in_class
JOIN
    Student ON Student_in_class.student = Student.id;
```| first\_name | last\_name   | class | student\_count\_in\_class |
| ----------- | ------------ | ----- | ------------------------- |
| Egor        | Belyakov     | 1     | 7                         |
| Artyom      | Nikitin      | 1     | 7                         |
| Innokentij  | Nekrasov     | 1     | 7                         |
| Semyon      | Voronov      | 1     | 7                         |
| Angelina    | Lazareva     | 1     | 7                         |
| Margarita   | Kabanova     | 1     | 7                         |
| Vera        | Evseeva      | 1     | 7                         |
| Denis       | Makarov      | 2     | 7                         |
| Arina       | SHarapova    | 2     | 7                         |
| Viktoriya   | Sergeeva     | 2     | 7                         |
| Vasilij     | Krasilnikov  | 2     | 7                         |
| Timur       | Rusakov      | 2     | 7                         |
| Gleb        | Nesterov     | 2     | 7                         |
| Elizaveta   | SHilova      | 2     | 7                         |
| Kirill      | SHubin       | 3     | 8                         |
| YUrij       | Markov       | 3     | 8                         |
| Grigorij    | Kolobov      | 3     | 8                         |
| Semyon      | Trofimov     | 3     | 8                         |
| Valentina   | SHarova      | 3     | 8                         |
| Larisa      | Savina       | 3     | 8                         |
| Galina      | Orekhova     | 3     | 8                         |
| Vasilij     | Ustinov      | 3     | 8                         |
| Timur       | Subbotin     | 4     | 9                         |
| Roman       | SHilov       | 4     | 9                         |
| Danila      | Osipov       | 4     | 9                         |
| Arina       | Silina       | 4     | 9                         |
| Nadezhda    | Zaharova     | 4     | 9                         |
| Larisa      | SHCHerbakova | 4     | 9                         |
| Aleksandra  | Belozyorova  | 4     | 9                         |
| Natalya     | Davydova     | 4     | 9                         |
| Mariya      | Fadeeva      | 4     | 9                         |
| Gennadij    | Ovchinnikov  | 5     | 8                         |
| Stanislav   | Lazarev      | 5     | 8                         |
| Viktoriya   | Makarova     | 5     | 8                         |
| Nataliya    | Myasnikova   | 5     | 8                         |
| Valentina   | Sazonova     | 5     | 8                         |
| Valeriya    | Lapina       | 5     | 8                         |
| ZHanna      | Fokina       | 5     | 8                         |
| Anna        | Kulagina     | 5     | 8                         |
| Ilya        | Filippov     | 6     | 11                        |
| Svyatoslav  | Muravyov     | 6     | 11                        |
| Denis       | Mel          | 6     | 11                        |
| Matvej      | Bogdanov     | 6     | 11                        |
| YUliya      | Avdeeva      | 6     | 11                        |
| Elizaveta   | Samojlova    | 6     | 11                        |
| Polina      | Sidorova     | 6     | 11                        |
| Lyubov      | Zaharova     | 6     | 11                        |
| Ilya        | Alekseev     | 6     | 11                        |
| Matvej      | YAkushev     | 6     | 11                        |
| Svyatoslav  | Tarasov      | 6     | 11                        |
| Nina        | Odincova     | 7     | 10                        |
| Boris       | Simonov      | 7     | 10                        |
| Dmitrij     | Trofimov     | 7     | 10                        |
| YAkov       | Rozhkov      | 7     | 10                        |
| Fyodor      | Drozdov      | 7     | 10                        |
| Gleb        | Strelkov     | 7     | 10                        |
| Angelina    | Lukina       | 7     | 10                        |
| Valeriya    | Novikova     | 7     | 10                        |
| Grigorij    | Kapustin     | 7     | 10                        |
| Vitalij     | Panfilov     | 7     | 10                        |
| Anna        | Osipova      | 8     | 11                        |
| Georgij     | Baranov      | 8     | 11                        |
| YUliya      | Vishnyakova  | 8     | 11                        |
| Valentina   | Bolshakova   | 8     | 11                        |
| Leonid      | Kryukov      | 8     | 11                        |
| Vladislav   | Cvetkov      | 8     | 11                        |
| Lyubov      | Borisova     | 8     | 11                        |
| Anfisa      | Kalashnikova | 8     | 11                        |
| Snezhana    | Morozova     | 8     | 11                        |
| Kristina    | Myasnikova   | 8     | 11                        |
| Kristina    | Smirnova     | 8     | 11                        |
| Vyacheslav  | Eliseev      | 9     | 13                        |
| Ivan        | Efremov      | 9     | 13                        |
| Anatolij    | ZHdanov      | 9     | 13                        |
| Georgij     | Noskov       | 9     | 13                        |
| Artyom      | Sergeev      | 9     | 13                        |
| Arina       | Evseeva      | 9     | 13                        |
| Angelina    | Voroncova    | 9     | 13                        |
| Ekaterina   | Ustinova     | 9     | 13                        |
| Raisa       | Lapina       | 9     | 13                        |
| Leonid      | Ignatov      | 9     | 13                        |
| Snezhana    | Seliverstova | 9     | 13                        |
| Semyon      | Biryukov     | 9     | 13                        |
| Nikolaj     | Sokolov      | 9     | 13                        |### Что делает наша оконная функцияВыражение `PARTITION BY Student_in_class.class` разделяет все строки таблицы на партиции по полю `class`.
Так, для каждой из строк в оконную функцию будут подаваться только те строки таблицы, где поле `class`
совпадает с полем `class` в текущей строке.Функция `COUNT` же возвращает количество переданных в неё строк, тем самым мы и получаем сколько учащихся
учится в каждом из классов.## Порядок выполнения оконных функций в SELECTПри использовании оконных функций важно понимать, в какой последовательности они будут исполняться. Так, как мы
можем увидеть на схеме ниже, окна отрабатывают предпоследним шагом, уже после фильтрации и группировки, но
перед финальной сортировкой результатов выборки.![Очередь выполнения оконной функции в SELECT запросе](https://sql-academy.org/static/guidePage/windows-functions/query-order.png "Очередь выполнения оконной функции в SELECT запросе")## ЗаключениеВ этой статье мы кратко рассмотрели понятие оконных функций, их возможности и практическую пользу.
В следующих статьях мы более подробно рассмотрим каждый аспект оконных функций.И напоследок давайте проверим, все ли мы поняли:**Какое ключевое отличие между оконными функциями и агрегатными функциями с группировкой в SQL?**1) Оконные функции и агрегатные функции с группировкой выполняют одни и те же вычисления, но с использованием разного синтаксиса. — Оконные функции и агрегатные функции с группировкой имеют разную функциональность и не могут использоваться взаимозаменяемо.

2) **Правильный ответ:&#x20;**&#x41E;конные функции вычисляются для каждой строки независимо, возвращая результат в отдельный столбец. Агрегатные функции с группировкой в свою очередь группируют строки и применяются к сформированным группам. — Оконные функции предоставляют расчёты для каждой строки, учитывая набор строк (окно), связанный с текущей строкой, в то время как агрегатные функции с группировкой предоставляют один результат для каждой группы, созданной по критерию группировки.

3) В оконных функциях используется PARTITION BY, а в агрегатных функциях с группировкой — нет. — Хотя PARTITION BY действительно является особенностью оконных функций, ключевое отличие заключается в том, как функции применяются к данным (по строкам против групп).
````
