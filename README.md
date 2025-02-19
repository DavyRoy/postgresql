# Postgres

---

### Задание 1 - Введение в PostgreSQL

1. `Разверни контейнер с PostgreSQL.`
2. `Cоздай таблицу для хранения пользователей с полями: id (целочисленный, автоинкремент, PRIMARY KEY) name (строка) email (строка)`
3. `Вставь несколько пользователей в таблицу и сделай выборку всех данных.`

``` ~Решение~
1.
docker run --name postgres -e POSTGRES_PASSWORD=password -d postgres
2.
docker exec -it postgres psql -U postgres
  CREATE DATABASE testdb;
  CREATE TABLE users ( id SERIAL PRIMARY KEY, name VARCHAR(100), email VARCHAR(100) ); 
3.
  INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com'); 
  INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com');
  SELECT * FROM users;
  ....
   id | name  |       email       
----+-------+-------------------
    1 | Alice | alica@example.com
    2 | Bob   | bob@example.com
  (2 rows)
  ....
```

`Cкриншоты
![Cкриншот 1](ссылка на скриншот 1)`

---

### Задание 2 - Основные SQL-запросы

1. `Создание выборки данных из таблицы: Напиши запрос, который выбирает все записи из таблицы users, но только с полями name и email.`
2. `Выполнение различных операций: Вставь нового пользователя в таблицу. Обнови email для существующего пользователя. Удали одного пользователя по имени.`
3. `Работа с агрегатными функциями и сортировка данных: Напиши запрос, который подсчитывает количество пользователей в таблице. Напиши запрос, который сортирует пользователей по имени в обратном порядке.`

``` ~Решение~
1.
SELECT name, email FROM users;
   name  |       email       
-------+-------------------
  Alice | alica@example.com
  Bob   | bob@example.com
  Tom   | tom@example.com
  Max   | max@example.com
  (4 rows)
2.
INSERT INTO users (name, email) VALUES ('Alex', 'alex@example.com');
  INSERT 0 1
UPDATE users SET email = 'alicenew@example.com' WHERE name = 'Alice';
  UPDATE 1
DELETE FROM users WHERE name = 'Tom';
  DELETE 1
3.
SELECT COUNT(*) FROM users;
   count 
-------
      4
  (1 row)
SELECT * FROM users ORDER BY name DESC;
   id | name  |        email         
----+-------+----------------------
    4 | Max   | max@example.com
    2 | Bob   | bob@example.com
    1 | Alice | alicenew@example.com
    5 | Alex  | alex@example.com
  (4 rows)
....
```

`Cкриншоты
![Скриншот](ссылки на скриншот)`

---

### Задание 3 - Работа с отношениями между таблицами

1. `Создай таблицу заказов, связывая её с таблицей пользователей через внешний ключ.`
2. `Напиши запрос, который выбирает все заказы пользователя с конкретным ID`


``` ~Решение~
1.
CREATE TABLE orders ( order_id SERIAL PRIMARY KEY, user_id INT, order_date DATE, FOREIGN KEY (user_id) REFERENCES users(id) );
2.
INSERT INTO orders (user_id, order_date) VALUES (5, '2025-05-14');
INSERT INTO orders (user_id, order_date) VALUES (4, '2025-05-14');
INSERT INTO orders (user_id, order_date) VALUES (5, '2025-06-14');
SELECT * FROM orders WHERE user_id = 1;
....
order_id | user_id | order_date 
----------+---------+------------
        1 |       1 | 2025-02-14
        2 |       1 | 2025-05-14
(2 rows)
....
SELECT * FROM orders WHERE user_id = 5;
....
order_id | user_id | order_date 
----------+---------+------------
        5 |       5 | 2025-05-14
        6 |       5 | 2025-01-14
        7 |       5 | 2025-06-14
(3 rows)
....
```

`Cкриншоты
![docker-network.pmg](https://github.com/DavyRoy/docker/blob/main/docker-network/images/docker-network.png)`

---

### Задание 4 - Индексы и оптимизация запросов

1. `Создай индекс на поле email в таблице users.`
2. `Проанализируй производительность запроса с помощью EXPLAIN ANALYZE.`

``` ~Решение~
1.
CREATE INDEX idx_users_email ON users (email);

2.
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'bob@example.com';
....
                                    QUERY PLAN                                            
-------------------------------------------------------------------------------------------------
 Seq Scan on users  (cost=0.00..1.06 rows=1 width=440) (actual time=0.026..0.029 rows=1 loops=1)
   Filter: ((email)::text = 'bob@example.com'::text)
   Rows Removed by Filter: 4
 Planning Time: 0.156 ms
 Execution Time: 0.054 ms
(5 rows)
....
```

`Скриншоты
![Скриншот ](ссылка на скриншот)`

---

### Задание 5 - Бэкапы и восстановление данных

1. `Сделай бэкап своей базы данных.`
2. `Восстанови базу данных в новый контейнер PostgreSQL.`

``` ~Решение~
1.
docker exec -t postgres pg_dump -U postgres -d testdb > /home/sergey/de
vops/postgresql/backup.sql
....
--
-- PostgreSQL database dump
--

-- Dumped from database version 17.2 (Debian 17.2-1.pgdg120+1)
-- Dumped by pg_dump version 17.2 (Debian 17.2-1.pgdg120+1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET transaction_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- PostgreSQL database dump complete
--
....
2.
docker exec -i postgres psql -U postgres -d postgres < /home/sergey/devops/postgresql/backup.sql
....
SET
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
....
```

`Скриншоты
![Скриншот ](ссылка на скриншот)`

---
