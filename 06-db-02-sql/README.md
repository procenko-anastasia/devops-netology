# Домашнее задание к занятию "6.2. SQL"

## Проценко Анастасия

## Задача 1

*Используя docker поднила инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.*

* Запустила контейнер с именем ***postgres_netology***
```bash
docker  run --name postgres_netology -e POSTGRES_PASSWORD=123 --mount=type=volume,src=vol1,dst=/home/vol1 --mount=type=volume,src=vol2,dst=/home/vol2 -d docker.io/library/postgres:12-alpine
```

* Подключаемся через CLI

```bash
psql -U postgres -d postgres
```

## Задача 2

*В БД из задачи 1:* 
- *создайте пользователя test-admin-user и БД test_db*  
```sql
postgres=# create database test_db; 
CREATE DATABASE
test_db=# create user test_admin_user;
CREATE ROLE
```
- *в БД test_db создайте таблицу orders и clients*
```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  наименование TEXT,
  цена INT
);

CREATE TABLE clients(
  id SERIAL PRIMARY KEY,
  фамилия TEXT,
  страна_проживания TEXT,
  заказ INT,
  CONSTRAINT fk_orders
    FOREIGN KEY (заказ)
    REFERENCES orders (id)
);

CREATE INDEX страна_проживания_idx ON clients(страна_проживания);
```
- *предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db*  
```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO test_admin_user; 
```
- *создайте пользователя test-simple-user*
```sql
CREATE USER test_simple_user;
```  
- *предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db*  
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO test_simple_user;
```

*Привожу:*
- *итоговый список БД после выполнения пунктов выше,*
```sql
test_db=# SELECT datname FROM pg_database;
  datname  
-----------
 postgres
 test_db
 template1
 template0
(4 rows)
```
- *описание таблиц (describe)*
```sql
test_db=# \d orders
                               Table "public.orders"
    Column    |  Type   | Collation | Nullable |              Default               
--------------+---------+-----------+----------+------------------------------------
 id           | integer |           | not null | nextval('orders_id_seq'::regclass)
 наименование | text    |           |          | 
 цена         | integer |           |          | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "fk_orders" FOREIGN KEY ("заказ") REFERENCES orders(id)

test_db=# \d clients
                                  Table "public.clients"
      Column       |  Type   | Collation | Nullable |               Default               
-------------------+---------+-----------+----------+-------------------------------------
 id                | integer |           | not null | nextval('clients_id_seq'::regclass)
 фамилия           | text    |           |          | 
 страна_проживания | text    |           |          | 
 заказ             | integer |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
    "страна_проживания_idx" btree ("страна_проживания")
Foreign-key constraints:
    "fk_orders" FOREIGN KEY ("заказ") REFERENCES orders(id)
```
- *SQL-запрос для выдачи списка пользователей с правами над таблицами test_db*
```sql
SELECT * from information_schema.table_privileges WHERE grantee LIKE 'test%';
```
- *список пользователей с правами над таблицами test_db*
```sql
test_db=# SELECT * from information_schema.table_privileges WHERE grantee LIKE 'test%';
 grantor  |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy 
----------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 postgres | test_admin_user  | test_db       | public       | orders     | INSERT         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | orders     | SELECT         | NO           | YES
 postgres | test_admin_user  | test_db       | public       | orders     | UPDATE         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | orders     | DELETE         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | orders     | TRUNCATE       | NO           | NO
 postgres | test_admin_user  | test_db       | public       | orders     | REFERENCES     | NO           | NO
 postgres | test_admin_user  | test_db       | public       | orders     | TRIGGER        | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | INSERT         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | SELECT         | NO           | YES
 postgres | test_admin_user  | test_db       | public       | clients    | UPDATE         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | DELETE         | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | TRUNCATE       | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | REFERENCES     | NO           | NO
 postgres | test_admin_user  | test_db       | public       | clients    | TRIGGER        | NO           | NO
 postgres | test_simple_user | test_db       | public       | orders     | INSERT         | NO           | NO
 postgres | test_simple_user | test_db       | public       | orders     | SELECT         | NO           | YES
 postgres | test_simple_user | test_db       | public       | orders     | UPDATE         | NO           | NO
 postgres | test_simple_user | test_db       | public       | orders     | DELETE         | NO           | NO
 postgres | test_simple_user | test_db       | public       | clients    | INSERT         | NO           | NO
 postgres | test_simple_user | test_db       | public       | clients    | SELECT         | NO           | YES
 postgres | test_simple_user | test_db       | public       | clients    | UPDATE         | NO           | NO
 postgres | test_simple_user | test_db       | public       | clients    | DELETE         | NO           | NO
(22 rows)
```

## Задача 3

*Используя SQL синтаксис - наполнила таблицы следующими тестовыми данными:*

*Таблица orders*
  
```sql
INSERT INTO orders (наименование, цена)
  VALUES
  ('Шоколад', 10),
  ('Принтер', 3000),
  ('Книга',   500),
  ('Монитор', 7000),
  ('Гитара',  4000);
```

*Таблица clients*

```sql
INSERT INTO clients (фамилия, страна_проживания)
  VALUES
  ('Иванов Иван Иванович', 'USA'),
  ('Петров Петр Петрович', 'Canada'),
  ('Иоганн Себастьян Бах', 'Japan'),
  ('Ронни Джеймс Дио', 'Russia'),
  ('Ritchie Blackmore', 'Russia');
```

*Используя SQL синтаксис:*
- *вычислила количество записей для каждой таблицы* 
- *привела в ответе:*
    - *запросы* 
    - *результаты их выполнения.*

```sql
test_db=# SELECT COUNT(id) FROM orders;
 count 
-------
     5
(1 row)

test_db=# SELECT COUNT(id) FROM clients;
 count 
-------
     5
(1 row)
```

## Задача 4

*Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.*

*Используя foreign keys связала записи из таблиц, согласно таблице:*

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

*Привела SQL-запросы для выполнения данных операций.*    
```sql
postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# UPDATE clients SET заказ=3 WHERE id=1;
UPDATE 1
test_db=# UPDATE clients SET заказ=4 WHERE id=2;
UPDATE 1
test_db=# UPDATE clients SET заказ=5 WHERE id=3;
UPDATE 1
```

*Привела SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.*    
```sql
test_db=# SELECT * FROM clients WHERE заказ IS NOT NULL;
 id |       фамилия        | страна_проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)
```
## Задача 5

*Получила полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).*

* Результат:
```sql
test_db=# EXPLAIN SELECT * FROM clients WHERE заказ IS NOT NULL;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)
```

Выбран план простого последовательного сканирования. 

* Приблизительная стоимость запуска (0.00). Это время, которое проходит, прежде чем начнётся этап вывода данных, например для сортирующего узла это время сортировки.

* Приблизительная общая стоимость (18.10). Она вычисляется в предположении, что узел плана выполняется до конца, то есть возвращает все доступные строки.

* Ожидаемое число строк (806), которое должен вывести этот узел плана. При этом так же предполагается, что узел выполняется до конца.

* Ожидаемый средний размер строк (72), выводимых этим узлом плана (в байтах).

## Задача 6

*Создаем бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).*

```bash
pg_dump -U postgres -Fc test_db -p 5432 > /var/tmp/test_db.dump
```

*Остановите контейнер с PostgreSQL (но не удаляйте volumes).*

```bash
Anastasia@192 ~ % docker ps
CONTAINER ID   IMAGE                COMMAND                  CREATED             STATUS             PORTS      NAMES
d91be14e1315   postgres:12-alpine   "docker-entrypoint.s…"   3 hours ago         Up 3 hours         5432/tcp   postgres_netology
Anastasia@192 ~ % docker stop d91be14e1315
d91be14e1315
Anastasia@192 ~ % docker ps               
CONTAINER ID   IMAGE                COMMAND                  CREATED             STATUS             PORTS      NAMES
Anastasia@192 ~ % docker rm d91be14e1315
d91be14e1315
```

*Подняла новый пустой контейнер с PostgreSQL.*

Заходим в контейнер:  
```bash
Anastasia@192 ~ % docker ps             
CONTAINER ID   IMAGE                COMMAND                  CREATED             STATUS             PORTS      NAMES
404f231f00bb   postgres:12-alpine   "docker-entrypoint.s…"   About an hour ago   Up About an hour   5432/tcp   postgres_netology_2
Anastasia@192 ~ % docker exec -it 404f231f00bb sh
/ # su - postgres
404f231f00bb:~$ 
```

*Восстанавливаю БД test_db в новом контейнере.*  

* Заходим в **psql**, проверяем что лишних баз нет,

```bash
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```

* Восстановление через **pg_restore**
```bash
pg_restore -C -d postgres /var/tmp/test_db.dump
```

* Заходим в **psql**
```sql
404f231f00bb:~$ psql
psql (12.10)
Type "help" for help.

postgres=# \l
                                    List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |      Access privileges       
-----------+----------+----------+------------+------------+------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                 +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                 +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                +
           |          |          |            |            | postgres=CTc/postgres       +
           |          |          |            |            | test_admin_user=CTc/postgres
(4 rows)
postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".
test_db=# SELECT * FROM clients;
 id |       фамилия        | страна_проживания | заказ 
----+----------------------+-------------------+-------
  4 | Ронни Джеймс Дио     | Russia            |      
  5 | Ritchie Blackmore    | Russia            |      
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(5 rows)

test_db=# SELECT * FROM orders;
 id | наименование | цена 
----+--------------+------
  1 | Шоколад      |   10
  2 | Принтер      | 3000
  3 | Книга        |  500
  4 | Монитор      | 7000
  5 | Гитара       | 4000
(5 rows)
```
