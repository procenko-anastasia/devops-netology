# Домашнее задание к занятию "6.3. MySQL"

## Проценко Анастасия

## Задача 1

*Используя docker поднимаем инстанс MySQL (версию 8). Данные БД сохраняем в volume.*. 
Создаём файл **docker-compose.yml**:
```yml
version: "3.8"

services:

  db:
    image: mysql:8-oracle
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - db-volume:/var/lib/mysql

    ports:
      - 3306:3306

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080

volumes:
  db-volume:
```

Поднимаем инстанс через 
```bash
docker-compose up
```

Заходим в MySQL и проверяем его работоспособность:
```bash
Anastasia@192 ~ % docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                               NAMES
a7273c4b64ed   adminer          "entrypoint.sh docke…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->8080/tcp              netology-adminer-1
6dc3a7ba6f2c   mysql:8-oracle   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:3306->3306/tcp, 33060/tcp   netology-db-1
Anastasia@192 ~ % docker exec -it 6dc3a7ba6f2c mysql -h db -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.28 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
----

*Изучаем [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и восстанавливаем из него.*

Копируем бэкап внутрь контейнера:
```bash
docker cp test_dump.sql 6dc3a7ba6f2c:/var/tmp/test_dump.sql
```

Создаём пустую базу данных для бэкапа:
```
mysql> CREATE DATABASE testdb;
Query OK, 1 row affected (0.01 sec)
```

Заливаем дамп в БД:
```
Anastasia@192 Netology % docker exec -it 6dc3a7ba6f2c bash
bash-4.4# mysql -h db -u root -p testdb < /var/tmp/test_dump.sql
Enter password: 
```
----
*Переходим в управляющую консоль `mysql` внутри контейнера.*
```
bash-4.4# mysql -h db -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.28 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
----
*Используя команду `\h` получили список управляющих команд.*
```
mysql> \h

For information about MySQL products and services, visit:
   http://www.mysql.com/
For developer information, including the MySQL Reference Manual, visit:
   http://dev.mysql.com/
To buy MySQL Enterprise support, training, or other products, visit:
   https://shop.mysql.com/

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.
resetconnection(\x) Clean session context.
query_attributes Sets string parameters (name1 value1 name2 value2 ...) for the next query to pick up.

For server side help, type 'help contents'

mysql> 
```
----
*Найшли команду для выдачи статуса БД и **привели в ответе** из ее вывода версию сервера БД.*
```
mysql> \s
--------------
mysql  Ver 8.0.28 for Linux on aarch64 (MySQL Community Server - GPL)
```
----
*Подключились к восстановленной БД и получили список таблиц из этой БД.*
```
mysql> USE testdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+------------------+
| Tables_in_testdb |
+------------------+
| orders           |
+------------------+
1 row in set (0.01 sec)
```
----
***Привели в ответе** количество записей с `price` > 300.*
```
mysql> SELECT COUNT(*) FROM orders WHERE price > 300;
+----------+
| COUNT(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)
```

## Задача 2

*Создали пользователя test в БД c паролем test-pass, используя:*
- *плагин авторизации mysql_native_password*
- *срок истечения пароля - 180 дней*
- *количество попыток авторизации - 3*
- *максимальное количество запросов в час - 100*
- *аттрибуты пользователя:*
    - *Фамилия "Pretty"*
    - *Имя "James"*
```sql
mysql> CREATE USER 'test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'test-pass'
    -> REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 100
    -> PASSWORD EXPIRE INTERVAL 180 DAY
    -> FAILED_LOGIN_ATTEMPTS 3
    -> ATTRIBUTE '{"fname": "James", "lname": "Pretty"}'
    -> ;
Query OK, 0 rows affected (0.02 sec)
```
----
*Предоставели привелегии пользователю `test` на операции SELECT базы `test_db`.*
```sql
mysql> GRANT SELECT ON testdb.* TO 'test'@'localhost';
Query OK, 0 rows affected, 1 warning (0.16 sec)
```
----    
*Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получили данные по пользователю `test` и 
**привели в ответе к задаче**.*
```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE user='test';
+------+-----------+---------------------------------------+
| USER | HOST      | ATTRIBUTE                             |
+------+-----------+---------------------------------------+
| test | localhost | {"fname": "James", "lname": "Pretty"} |
+------+-----------+---------------------------------------+
1 row in set (0.00 sec)
```

## Задача 3

*Установили профилирование `SET profiling = 1`.
Изучили вывод профилирования команд `SHOW PROFILES;`.*

Устанавливаем профилирование:
```sql
mysql> SET profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
Выводим пару последних **SELECT**
```sql
mysql> SHOW PROFILES;
+----------+------------+--------------------------------------------------+
| Query_ID | Duration   | Query                                            |
+----------+------------+--------------------------------------------------+
|        1 | 0.00194000 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES |
|        2 | 0.00185600 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES |
|        3 | 0.00121825 | SELECT * FROM orders WHERE price > 300           |
+----------+------------+--------------------------------------------------+
3 rows in set, 1 warning (0.01 sec)
```
----
*Исследуем, какой `engine` используется в таблице БД `testdb` и **приведем в ответе**.*
```sql
mysql> SELECT ENGINE, TABLE_NAME FROM information_schema.tables WHERE TABLE_NAME='orders';
+--------+------------+
| ENGINE | TABLE_NAME |
+--------+------------+
| InnoDB | orders     |
+--------+------------+
1 row in set (0.00 sec)

```

*Изменяем `engine` и **приведем время выполнения и запрос на изменения из профайлера в ответе**:*
- *на `MyISAM`*
```sql
mysql> ALTER TABLE orders ENGINE=MyISAM;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                              |
+----------+------------+------------------------------------------------------------------------------------+
|        1 | 0.00194000 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES                                   |
|        2 | 0.00185600 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES                                   |
|        3 | 0.00121825 | SELECT * FROM orders WHERE price > 300                                             |
|        4 | 0.00347500 | SELECT ENGINE, TABLE_NAME FROM information_schema.tables WHERE TABLE_NAME='orders' |
|        5 | 0.04932350 | ALTER TABLE orders ENGINE=MyISAM                                                   |
+----------+------------+------------------------------------------------------------------------------------+
5 rows in set, 1 warning (0.01 sec)
```
- *на `InnoDB`*
```sql
mysql> ALTER TABLE orders ENGINE=InnoDB;
Query OK, 5 rows affected (0.05 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                              |
+----------+------------+------------------------------------------------------------------------------------+
|        1 | 0.00194000 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES                                   |
|        2 | 0.00185600 | SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES                                   |
|        3 | 0.00121825 | SELECT * FROM orders WHERE price > 300                                             |
|        4 | 0.00347500 | SELECT ENGINE, TABLE_NAME FROM information_schema.tables WHERE TABLE_NAME='orders' |
|        5 | 0.04932350 | ALTER TABLE orders ENGINE=MyISAM                                                   |
|        6 | 0.04755525 | ALTER TABLE orders ENGINE=InnoDB                                                   |
+----------+------------+------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```

## Задача 4 

*Изучаем файл `my.cnf` в директории /etc/mysql.*

*Изменяем его согласно ТЗ (движок InnoDB):*
- *Скорость IO важнее сохранности данных*
- *Нужна компрессия таблиц для экономии места на диске*
- *Размер буффера с незакомиченными транзакциями 1 Мб*
- *Буффер кеширования 30% от ОЗУ*
- *Размер файла логов операций 100 Мб*

*Приводим в ответе измененный файл `my.cnf`.*
```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M

# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
# default-authentication-plugin=mysql_native_password
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid

#Скорость IO важнее сохранности данных
innodb_flush_log_at_trx_commit  = 2

#Компрессия таблиц для экономии места на диске (1 файл на таблицу)
innodb_file_per_table       = 1

#Размер буффера с незакомиченными транзакциями 1 Мб
innodb_log_buffer_size      = 1M

#Буффер кеширования 30% от ОЗУ
innodb_buffer_pool_size     = 4700M

#Размер файла логов операций 100 Мб
innodb_log_file_size        = 100M

!includedir /etc/mysql/conf.d/
```
---
