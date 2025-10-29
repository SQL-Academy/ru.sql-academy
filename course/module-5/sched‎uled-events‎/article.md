---
meta:
    title: 'Планировщик событий в SQL: MySQL EVENT и PostgreSQL pg_cron'
    description: 'Полное руководство по созданию автоматических задач в MySQL и PostgreSQL. Научитесь использовать EVENT scheduler и pg_cron для очистки данных, обновления статистики и генерации отчётов по расписанию. Примеры кода и практические советы.'
---

<DBMSSwitcher />

# Запланированные события (Events)

В реальных приложениях часто возникает необходимость выполнять определённые действия автоматически и по расписанию: очищать старые записи, обновлять статистику, формировать отчёты. Для таких задач SQL предоставляет механизм **запланированных событий**.

> **Событие** — это задача, которую база данных запускает сама по расписанию. Вы настраиваете — оно работает автоматически.

<MySQLOnly>

События в MySQL похожи на планировщик задач в операционной системе: вы создаёте задачу один раз, а база данных выполняет её автоматически по расписанию.

</MySQLOnly>

<PostgreSQLOnly>

В PostgreSQL для автоматического выполнения задач используется расширение **pg_cron**. Это расширение позволяет планировать SQL-команды с использованием синтаксиса cron (как в Unix-системах).

</PostgreSQLOnly>

## Когда он пригодится

Запланированные события помогают автоматизировать следующие задачи:

-   **Очистка данных**: удаление устаревших записей логов или временных данных
-   **Обновление статистики**: пересчёт агрегированных данных для аналитики
-   **Генерация отчётов**: автоматическое формирование периодических отчётов
-   **Резервное копирование**: создание копий важных данных

## Как включить планировщик

<MySQLOnly>

Прежде чем создавать события, необходимо убедиться, что планировщик событий включён:

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

Если планировщик выключен, включите его:

```sql
SET GLOBAL event_scheduler = ON;
```

</MySQLOnly>

<PostgreSQLOnly>

Для использования запланированных задач в PostgreSQL нужно установить расширение pg_cron:

```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
```

> **Важно:** Расширение pg_cron может требовать прав суперпользователя и дополнительной настройки PostgreSQL. В облачных сервисах (AWS RDS, Azure) оно может быть уже предустановлено.

</PostgreSQLOnly>

## Создание одноразового события

Начнём с самого простого — события, которое выполнится один раз в определённое время:

<MySQLOnly>

```sql-executable
CREATE EVENT cleanup_old_logs
ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 DAY
DO
    DELETE FROM logs WHERE created_at < NOW() - INTERVAL 30 DAY;
```

Это событие удалит записи логов старше 30 дней через 24 часа после создания события.

**Разберём синтаксис:**

-   `CREATE EVENT cleanup_old_logs` — создаём событие с именем `cleanup_old_logs`
-   `ON SCHEDULE AT` — указываем, когда событие должно выполниться
-   `CURRENT_TIMESTAMP + INTERVAL 1 DAY` — время выполнения (через 1 день)
-   `DO` — код, который нужно выполнить (любой SQL-оператор)

</MySQLOnly>

<PostgreSQLOnly>

```sql
SELECT cron.schedule(
    'cleanup_old_logs',
    '0 3 * * *',
    'DELETE FROM logs WHERE created_at < NOW() - INTERVAL ''30 days'''
);
```

Это событие будет выполняться каждый день в 3:00 утра и удалять записи логов старше 30 дней.

**Разберём синтаксис:**

-   `cron.schedule()` — функция для создания запланированной задачи
-   `'cleanup_old_logs'` — имя задачи
-   `'0 3 * * *'` — расписание в формате cron (минута час день месяц день_недели)
-   Последний параметр — SQL-команда для выполнения

**Формат cron расписания:**

![Формат cron расписания](https://sql-academy.org/static/guidePage/scheduled-events/cron_schedule_ru.png 'Формат cron расписания')

</PostgreSQLOnly>

## Создание повторяющегося события

Чаще всего события нужно запускать периодически — каждый день, час или минуту:

<MySQLOnly>

```sql-executable
CREATE EVENT update_statistics
ON SCHEDULE EVERY 1 HOUR
DO
BEGIN
    UPDATE product_stats SET
        total_sales = (SELECT SUM(amount) FROM orders WHERE product_id = product_stats.product_id),
        last_updated = NOW();
END;
```

Это событие будет обновлять статистику продаж каждый час.

**Разберём синтаксис:**

-   `ON SCHEDULE EVERY 1 HOUR` — выполнять каждый час
-   `BEGIN ... END` — блок из нескольких SQL-операторов

**Варианты интервалов:**

-   `EVERY 1 MINUTE` — каждую минуту
-   `EVERY 1 HOUR` — каждый час
-   `EVERY 1 DAY` — каждый день
-   `EVERY 1 WEEK` — каждую неделю
-   `EVERY 1 MONTH` — каждый месяц
-   `EVERY 30 SECOND` — каждые 30 секунд

</MySQLOnly>

<PostgreSQLOnly>

```sql
SELECT cron.schedule(
    'update_statistics_hourly',
    '0 * * * *',
    $$
    UPDATE product_stats SET
        total_sales = (SELECT SUM(amount) FROM orders WHERE product_id = product_stats.product_id),
        last_updated = NOW()
    $$
);
```

Это событие будет обновлять статистику продаж каждый час (в начале каждого часа).

**Примеры расписаний:**

-   `'*/5 * * * *'` — каждые 5 минут
-   `'0 * * * *'` — каждый час (в начале часа)
-   `'0 0 * * *'` — каждый день в полночь
-   `'0 0 * * 0'` — каждое воскресенье в полночь
-   `'0 9 1 * *'` — первого числа каждого месяца в 9:00

</PostgreSQLOnly>

## Событие с ограниченным сроком действия

Иногда нужно, чтобы событие работало только в определённый период:

<MySQLOnly>

```sql-executable
CREATE EVENT seasonal_discount
ON SCHEDULE EVERY 1 DAY
STARTS '2025-12-01 00:00:00'
ENDS '2025-12-31 23:59:59'
DO
    UPDATE products SET price = price * 0.9 WHERE category = 'seasonal';
```

Это событие будет применять скидку 10% на сезонные товары каждый день в течение декабря 2025 года.

**Новые элементы:**

-   `STARTS` — начало периода действия события
-   `ENDS` — конец периода действия события

После указанной даты событие автоматически прекратит выполняться.

</MySQLOnly>

<PostgreSQLOnly>

В pg_cron нет встроенной поддержки автоматического завершения задач, но можно включить проверку даты в саму команду:

```sql
SELECT cron.schedule(
    'seasonal_discount',
    '0 0 * * *',
    $$
    UPDATE products
    SET price = price * 0.9
    WHERE category = 'seasonal'
      AND CURRENT_DATE BETWEEN '2025-12-01' AND '2025-12-31'
    $$
);
```

Альтернативно, можно создать задачу на удаление события в конце периода:

```sql
SELECT cron.schedule(
    'remove_seasonal_discount',
    '0 0 1 1 *',  -- 1 января в полночь
    $$SELECT cron.unschedule('seasonal_discount')$$
);
```

</PostgreSQLOnly>

## Просмотр существующих событий

<MySQLOnly>

Чтобы увидеть все созданные события:

```sql
SHOW EVENTS;
```

Для просмотра событий в конкретной базе данных:

```sql
SHOW EVENTS FROM your_database_name;
```

</MySQLOnly>

<PostgreSQLOnly>

Чтобы увидеть все запланированные задачи:

```sql
SELECT * FROM cron.job;
```

Это вернёт таблицу со всеми задачами, включая их расписание и команды.

Для просмотра истории выполнения задач:

```sql
SELECT * FROM cron.job_run_details
ORDER BY start_time DESC
LIMIT 10;
```

</PostgreSQLOnly>

## Управление событиями

<MySQLOnly>

**Временное отключение события:**

```sql
ALTER EVENT cleanup_old_logs DISABLE;
```

**Включение события:**

```sql
ALTER EVENT cleanup_old_logs ENABLE;
```

**Изменение расписания события:**

```sql
ALTER EVENT cleanup_old_logs
ON SCHEDULE EVERY 2 HOUR;
```

**Удаление события:**

```sql
DROP EVENT IF EXISTS cleanup_old_logs;
```

</MySQLOnly>

<PostgreSQLOnly>

**Удаление запланированной задачи:**

```sql
SELECT cron.unschedule('cleanup_old_logs');
```

Или по ID задачи:

```sql
SELECT cron.unschedule(42);  -- где 42 - это jobid из таблицы cron.job
```

**Изменение задачи:**

В pg_cron нельзя изменить существующую задачу напрямую. Нужно удалить старую и создать новую:

```sql
-- Удаляем старую
SELECT cron.unschedule('cleanup_old_logs');

-- Создаём новую с обновлённым расписанием
SELECT cron.schedule(
    'cleanup_old_logs',
    '0 */2 * * *',  -- Каждые 2 часа
    'DELETE FROM logs WHERE created_at < NOW() - INTERVAL ''30 days'''
);
```

</PostgreSQLOnly>

## Важные моменты при работе с событиями

<MySQLOnly>

1. **Права доступа**: Для создания событий нужна привилегия `EVENT`.

2. **Временная зона**: События выполняются по временной зоне сервера базы данных.

3. **Производительность**: Избегайте создания событий со слишком коротким интервалом (каждую минуту), это может повлиять на производительность.

</MySQLOnly>

<PostgreSQLOnly>

1. **Права доступа**: Для использования pg_cron обычно требуются права суперпользователя или специальная настройка.

2. **Временная зона**: Задачи pg_cron выполняются по временной зоне PostgreSQL (можно проверить через `SHOW timezone;`).

3. **Производительность**: Pg_cron проверяет расписание каждую секунду, поэтому минимальная точность — 1 минута.

4. **Логирование**: Все выполнения задач сохраняются в таблице `cron.job_run_details`, что полезно для отладки.

</PostgreSQLOnly>

## Самопроверка

Какой минимальный интервал можно использовать для повторяющихся событий?
Запланированные события — это мощный инструмент для автоматизации рутинных задач в базе данных. Они помогают поддерживать чистоту данных, обновлять статистику и выполнять регламентные операции без участия разработчиков! 🚀
