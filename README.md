# Домашнее задание к занятию "Индексы" - Голиков Алексей

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение 1

```
select round (sum(index_length) / sum(data_length) * 100) as '% index'
from INFORMATION_SCHEMA.TABLES;
```

### Эталонное решение 1
```
SELECT
   ROUND(SUM(data_lenght) / (SUM(data_lenght) + SUM(index_lenght)) * 100, 2) AS tables_percentage,
   ROUND(SUM(index_lenght) / (SUM(data_lenght) + SUM(index_lenght)) * 100, 2) AS index_percentage
FROM
   information_schema.tables
WHERE
   table_schema = 'sakila';
```


---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

### Решение 2

```
select distinct concat(c.last_name, ' ', c.first_name) as client, sum(p.amount) over (partition by c.customer_id) as pays
from payment p, customer c
where date(p.payment_date) = '2005-07-30' and p.customer_id = c.customer_id;
```

### Эталонное решение 2
```
SELECT
   CONCAT(c.last_name, ' ', c.first_name) AS full_name,
   SUM(p.amount) AS total_amount_per_customer_per_film
FROM
   payment p
JOIN
   rental r ON p.payment_date = r.rental_date
JOIN
   customer c ON r.customer_id = c.customer_id
JOIN
   inventory i ON r.inventory_id = i.inventory_id
JOIN
   film f ON i.film_id = f.film_id
WHERE
   p.payment_date >= '2005-07-30' AND p.payment_date < '2005-07-31'
GROUP BY
   c.customer_id, c.last_name, c.first_name;
```
---
