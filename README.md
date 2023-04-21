# Домашнее задание к занятию «Индексы»
## Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.
```
select sum(t.INDEX_LENGTH) /  sum(t.DATA_LENGTH) *100 as 'Отношение'  
from information_schema.TABLES t
```

![image](https://user-images.githubusercontent.com/121442297/233633472-81244b4a-81ec-4cba-916f-8708d2c2a897.png)


## Задание 2
Выполните explain analyze следующего запроса:
```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
перечислите узкие места;
оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
Ответ:
При проверке исполнения запроса выяснилось, что наибольшее время теряется на определении оконной функции. В данном запросе нет необходимости использовать таблицу film и вычленять из нее данные по столбцу f.title, т.к. это не дает никакой дополнительной информации для запроса. Удаление данной таблицы привело к снижению времени запроса с 4523,745 ms до 9,861 ms, при тех же результатах. В запросе происходит преобразование payment_date с datetime в date, что требует времени и приводит к невозможности использования индексов. Поменял запрос на вариант с between без преобразования, добавил индексы для payment_date и rental_date. Это привело снижение времени обработки запроса до 5,513 ms

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, rental r, customer c, inventory i
where p.payment_date between  '2005-07-30 00:00:00' and '2005-07-30 23:59:59' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

```
explain analyze
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, rental r, customer c, inventory i
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
# Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.
# Задание 3*
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.
Приведите ответ в свободной форме.

