1. Создать новый проект в Яндекс облако или на любых ВМ, например postgres2024-<yyyymmdd>, где
yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально)

Создаем виртуальную машину (ВМ) в Яндекс.Облаке на странице https://cloud.yandex.ru/docs/compute/quickstart/quick-create-linux

1.2 Генерируем ssh-key

Использовано приложение SSH-клиента PuTTY.
Сгенерированный ssh-ключ указываем в поле SSH-ключ при создании ВМ в Яндекс.Облаке.

1.3 Подключаемся к ВМ

Открываем консоль.

Результат подключения в консоли:
```bash
Using username "hw1".
Authenticating with public key "rsa-key-20251002"
Passphrase for key "rsa-key-20251002":
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-84-generic x86_64)
 
* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/pro
 
System information as of Fri Oct  3 07:53:10 UTC 2025

System load:  0.01               Processes:             133
Usage of /:   11.3% of 18.72GB   Users logged in:       0
Memory usage: 13%                IPv4 address for eth0: 10.130.0.16
Swap usage:   0%

* Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
just raised the bar for easy, resilient and secure K8s cluster deployment.

https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

* System restart required *
Last login: Fri Oct  3 07:51:24 2025 from 217.28.247.164
hw1@otus-hw-1:~$
```

2. Установка PostgreSQL
В консоли пишем команду:
```bash
sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql && sudo apt install unzip && sudo apt -y install mc
```
Далее в консоли выполним команду:
```bash
pg_lsclusters
```
Результат:
```bash
hw1@otus-hw-1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```
Установка PostgreSQL выполнена.

3. Подключение к PostgreSQL
3.1 В консоли выполним команду подключения к БД:
```bash
sudo -u postgres psql
```

3.2 В консоли выполним команду просмотра списка БД:
```bash
\l
```
Результат:
```bash
                                                 List of databases
   Name    |  Owner   | Encoding | Locale Provider | Collate |  Ctype  | Locale | ICU Rules |   Access privileges
-----------+----------+----------+-----------------+---------+---------+--------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           |
 template0 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |         |         |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |        |           | =c/postgres          +
           |          |          |                 |         |         |        |           | postgres=CTc/postgres
```

3.3 Создаем базу данных и переходим в нее 
```bash
postgres=# create database otus_hw1;
CREATE DATABASE
postgres=# \c otus_hw1
You are now connected to database "otus_hw1" as user "postgres".
otus_hw1=#
```

3.4 Открытие второй SSH-сессии и подключение к PostgreSQL
```bash
 sudo -u postgres psql -d otus_hw1
```

4. Работа с транзакциями PostgreSQL
Создаем таблицу и заполняем данными
```bash
otus_hw1=# create table pokupatel (id serial, name varchar(150), cur_amt int);
CREATE TABLE
otus_hw1=# insert into pokupatel (name,cur_amt) values ('Ivanov',1000);
INSERT 0 1
otus_hw1=# insert into pokupatel (name,cur_amt) values ('Petrov',2000);
INSERT 0 1
otus_hw1=# insert into pokupatel (name,cur_amt) values ('Sidorov',3000);
INSERT 0 1
```
Результат:
```bash
otus_hw1=# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  1 | Ivanov |    1000
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
(3 rows)
```

5. Работа с уровнями изоляции
5.1 Уровень изоляции read committed - чтение зафиксированных данных

- В консоли 1 проверяем текущий уровень изоляции с помощью команды:
```bash
otus_hw1=# SHOW TRANSACTION ISOLATION LEVEL;
```
Результат:
```bash
 transaction_isolation
-----------------------
 read committed
(1 row)
```
- В консоли 1 открываем новую транзакцию
```bash
otus_hw1=# begin;
BEGIN
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  1 | Ivanov |    1000
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
(3 rows)

otus_hw1=*#
```
- В консоли 2 открываем новую транзакцию и изменяем данные в таблице
```bash
otus_hw1=# begin;
BEGIN
otus_hw1=*# update pokupatel set cur_amt = 5000 where id = 1;
UPDATE 1
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
(3 rows)
```
Результат:
```bash
Во второй сессии (консоль 2) видим, что у клиента с id=1 изменилась сумма на 5000.
```
В первой сессии (консоль 1) данные не изменились
```bash
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    1000
(3 rows)
```
- Завершаем транзакцию в консоли 2
```bash
otus_hw1=*# commit;
COMMIT
otus_hw1=# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
(3 rows)

otus_hw1=#
```
- Проверяем консоль 1
```bash
otus_hw1=*# commit;
COMMIT
otus_hw1=# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
(3 rows)

otus_hw1=#
```
Результат:
```bash
После завершения транзакции видим изменения во всех открытых сессиях.
В транзакции, работающей на этом уровне, запрос SELECT видит только те данные, которые были зафиксированы до начала запроса; он никогда не увидит незафиксированных данных или изменений, внесённых в процессе выполнения запроса параллельными транзакциями.
```
5.2 Уровень изоляции repeatable read - повторяемое чтение
- В консоли поменяем уровень изоляции БД на repeatable read с помощью команды:
```bash
otus_hw1=# begin;
BEGIN
otus_hw1=*# set transaction isolation level repeatable read;
SET
otus_hw1=*# show transaction isolation level;
 transaction_isolation
-----------------------
 repeatable read
(1 row)
```
- В консоли 1 добавляем новую запись в таблицу:
```bash
otus_hw1=*# insert into pokupatel (name, cur_amt) values ('Smirnov',4000);
INSERT 0 1
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
  4 | Smirnov |    4000
(4 rows)
```
Результат:
```bash
Новая запись с id = 4 добавлена.
```
- В консоли 2 выполняем запрос к таблице:
```bash
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
(3 rows)
```
Результат:
```bash
Новой записи нет.
```
- Завершаем транзакцию в консоли 1 с помощью commit:
```bash
otus_hw1=*# commit;
COMMIT
otus_hw1=# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
  4 | Smirnov |    4000
(4 rows)
```
- Теперь снова в консоли 2 повторяем запрос к таблице:
```bash
otus_hw1=*# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
(3 rows)
```
Результат:
```bash
Новой записи нет.
```
- Завершаем вторую транзацию и снова выполняем запрос к таблице:
```bash
otus_hw1=*# commit;
COMMIT
otus_hw1=# select * from pokupatel;
 id |     name      | cur_amt
----+---------------+---------
  2 | Petrov   |    2000
  3 | Sidorov     |    3000
  1 | Ivanov |    5000
  4 | Smirnov |    4000
(4 rows)

otus_hw1=#
```
Результат:
```bash
Новая запись с id = 4 отображается.
```

Вывод:
При уровне изоляции Repeatable Read видны только те данные, которые были зафиксированы до начала транзакции, но не видны незафиксированные данные и изменения, произведённые другими транзакциями в процессе выполнения данной транзакции.
