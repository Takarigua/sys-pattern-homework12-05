# Домашнее задание к занятию "`Индексы`" - `Марченко Вячеслав Игоревич`

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Ответ

1. ![Task1](https://github.com/Takarigua/sys-pattern-homework12-05/blob/13601278c31a4910074fa2b776cb859f6c7fea32/img/Task%201.png)


```
SELECT 
    table_schema, 
    SUM(index_length) AS total_index_size, 
    SUM(data_length) AS total_data_size,
    (SUM(index_length) / NULLIF(SUM(data_length), 0)) * 100 AS index_to_data_ratio
FROM 
    information_schema.TABLES 
GROUP BY 
    table_schema 
HAVING 
    table_schema = 'sakila';
```
---

### Задание 2

Выполните explain analyze следующего запроса:

select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Ответ

1. Возможная проблема с неэффективными соединениями при отсутствии индексов, а также применение функции DATE к колонке может повлечь за собой полное сканирование таблицы из-за невозможности использования индекса по дате. Делал несколько вариантов, но остановился на этом
3. ![Task2](https://github.com/Takarigua/sys-pattern-homework12-05/blob/865a8f36c1f50dbc2af3503c8f6f6145a0d93abf/img/Task%202.png)

```
EXPLAIN ANALYZE
SELECT DISTINCT
    CONCAT(c.last_name, ', ', c.first_name),
    SUM(p.amount) OVER (PARTITION BY c.customer_id)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
WHERE p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00';
```
