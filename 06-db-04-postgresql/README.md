# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql
***
```
mydb=# \l
                             List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | Access privileges 
-----------+-------+----------+------------+------------+-------------------
 mydb      | admin | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | admin | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | admin | UTF8     | en_US.utf8 | en_US.utf8 | =c/admin         +
           |       |          |            |            | admin=CTc/admin
 template1 | admin | UTF8     | en_US.utf8 | en_US.utf8 | =c/admin         +
           |       |          |            |            | admin=CTc/admin
(4 rows)

mydb=# \conninfo
You are connected to database "mydb" as user "admin" on host "127.0.0.1" at port "5432".

# Таблиц нет.
mydb=# \d
Did not find any relations.

mydb=# \d[S+] NAME

mydb=# \q
beketov@beketovs-MacBook-Pro 13 % 
```
***

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

***
```
mydb=# CREATE DATABASE test_database;
CREATE DATABASE

beketov@beketovs-MacBook-Pro backup % cat test_dump.sql| docker exec -i postgres13 psql -U admin -d test_database

beketov@beketovs-MacBook-Pro backup % psql -h 127.0.0.1 -d test_database -U admin                            
Password for user admin: 
psql (14.3, server 13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

test_database=# \d

             List of relations
 Schema |     Name      |   Type   | Owner 
--------+---------------+----------+-------
 public | orders        | table    | admin
 public | orders_id_seq | sequence | admin
(2 rows)

beketov@beketovs-MacBook-Pro backup % docker exec -it postgres13 /bin/bash
root@2cf6d06a8d54:/# psql -U admin -d test_database
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

test_database=# 

test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE


```
***

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
