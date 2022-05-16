# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

***
```
mysql> status
--------------
mysql  Ver 8.0.29 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:		15
Current database:	test_db
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.29 MySQL Community Server - GPL
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8mb4
Db     characterset:	utf8mb4
Client characterset:	latin1
Conn.  characterset:	latin1
UNIX socket:		/var/run/mysqld/mysqld.sock
Binary data as:		Hexadecimal
Uptime:			10 min 46 sec

Threads: 2  Questions: 88  Slow queries: 0  Opens: 183  Flush tables: 3  Open tables: 101  Queries per second avg: 0.136
--------------
```
```
mysql> SHOW TABLES;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)
```
```
mysql> SELECT * FROM orders WHERE price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
```
***

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

***
```
mysql> CREATE USER 'test' IDENTIFIED BY 'test-pass';
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER USER 'test' IDENTIFIED BY 'test-pass' WITH MAX_QUERIES_PER_HOUR 100 PASSWORD EXPIRE INTERVAL 180 DAY FAILED_LOGIN_ATTEMPTS 3;
Query OK, 0 rows affected (0.01 sec)

mysql> ALTER USER 'test' ATTRIBUTE '{"first_name":"James", "last_name":"Pretty"}';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT Select ON test_db.orders TO 'test';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+------+------------------------------------------------+
| USER | HOST | ATTRIBUTE                                      |
+------+------+------------------------------------------------+
| test | %    | {"last_name": "Pretty", "first_name": "James"} |
+------+------+------------------------------------------------+
1 row in set (0.00 sec)
```
***

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

***
```
mysql> SHOW TABLE STATUS WHERE Name = 'orders';
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| Name   | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation          | Checksum | Create_options | Comment |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| orders | InnoDB |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |            0 |         0 |              6 | 2022-05-16 20:32:45 | 2022-05-16 20:32:45 | NULL       | utf8mb4_0900_ai_ci |     NULL |                |         |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
1 row in set (0.03 sec)

mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.12 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW TABLE STATUS WHERE Name = 'orders';
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| Name   | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation          | Checksum | Create_options | Comment |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| orders | MyISAM |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |            0 |         0 |              6 | 2022-05-16 22:38:06 | 2022-05-16 20:32:45 | NULL       | utf8mb4_0900_ai_ci |     NULL |                |         |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
1 row in set (0.00 sec)
```
```
mysql> SHOW PROFILES;
+----------+------------+--------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                              |
+----------+------------+--------------------------------------------------------------------+
|        1 | 0.00113175 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test' |
|        2 | 0.00142200 | SHOW ENGINES                                                       |
|        3 | 0.02586800 | SHOW TABLE STATUS WHERE Name = 'test_db'                           |
|        4 | 0.00695350 | SHOW TABLES                                                        |
|        5 | 0.02599950 | SHOW TABLE STATUS WHERE Name = 'orders'                            |
|        6 | 0.12248250 | ALTER TABLE orders ENGINE = MyISAM                                 |
|        7 | 0.00418450 | SHOW TABLE STATUS WHERE Name = 'orders'                            |
+----------+------------+--------------------------------------------------------------------+
7 rows in set, 1 warning (0.00 sec)
```
Запрос с Query_ID = 5 - время при движке InnoDB, запрос с Query_ID = 7 - движок MyISAM. Запрос при движке MyISAM выполняется быстрее.
***

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

***
```
root@1c718969f4b3:/etc/mysql# cat my.cnf
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
# Скорость IO важнее сохранности данных
innodb_flush_method=O_DIRECT
innodb_flush_log_at_trx_commit=2
# Нужна компрессия таблиц для экономии места на диске
innodb_file_per_table=1
# Размер буффера с незакомиченными транзакциями 1 Мб
innodb_log_buffer_size=1M
# Буффер кеширования 30% от ОЗУ
innodb_buffer_pool_size=5G
# Размер файла логов операций 100 Мб
innodb_log_file_size=100M
```
***

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
