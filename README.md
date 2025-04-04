# Домашнее задание к занятию "`SQL. Часть 2`" - `Барышков Михаил`

## Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

---

## Решение 1

```sql
SELECT
 ROUND(SUM(index_length) / SUM(data_length + index_length) * 100, 2) AS "процентное отношение размера индексов к общему размеру (данные + индексы)",
 CONCAT(ROUND(SUM(data_length) / (1024 * 1024), 2), ' MB') AS "Общий размер таблиц",
 CONCAT(ROUND(SUM(index_length) / (1024 * 1024), 2), ' MB') AS "Общий размер индексов"
FROM
 information_schema.TABLES
WHERE
 table_schema = 'sakila';
```

<img src = "img/img1.png" width = 80%>

Этот запрос:

1. Считает общий размер данных (data_length) и общий размер индексов (index_length) для всех таблиц в схеме 'sakila'Вычисляет процентное отношение размера индексов к общему размеру (данные + индексы)

2. Также предоставляет информацию об общем размере таблиц и индексов в мегабайтах для наглядности

3. Результат покажет, какую часть от общего размера базы данных занимают индексы, что помогает оценить эффективность их использования.

---

## Задание 2

Выполните explain analyze следующего запроса:

```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

---

## Решение 2

```sql
EXPLAIN ANALYZE
SELECT
 DISTINCT concat(c.last_name, ' ', c.first_name),
 sum(p.amount) OVER (PARTITION BY c.customer_id,
 f.title)
FROM
 payment p,
 rental r,
 customer c,
 inventory i,
 film f
WHERE
 date(p.payment_date) = '2005-07-30'
 AND p.payment_date = r.rental_date
 AND r.customer_id = c.customer_id
 AND i.inventory_id = r.inventory_id;
```

-> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4628..4628 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4628..4628 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=2309..4468 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=2309..2368 rows=642000 loops=1)
                -> Stream results  (cost=21.6e+6 rows=15.4e+6) (actual time=30.4..1782 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=21.6e+6 rows=15.4e+6) (actual time=30.4..1539 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=20e+6 rows=15.4e+6) (actual time=25.9..1386 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=18.5e+6 rows=15.4e+6) (actual time=21.6..1207 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1.54e+6 rows=15.4e+6) (actual time=15.8..76.8 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.83 rows=15400) (actual time=7.59..26.5 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.83 rows=15400) (actual time=7.11..24.4 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=112 rows=1000) (actual time=7.65..8.09 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=1 rows=1) (actual time=0.00113..0.00167 rows=1.01 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.001 rows=1) (actual time=156e-6..173e-6 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.001 rows=1) (actual time=121e-6..139e-6 rows=1 loops=642000)


## Дополнительные задания (со звёздочкой*)

Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

*Приведите ответ в свободной форме.*
