1. В Яндекс облаке создан инстанс виртуальной машины с параметрами 
```
- Имя: hw2
- Внутренний FQDN: hw2.ru-central1.internal
- OS: Ubuntu 24.04 LTS
- CPU: 2 ( 50%	)
- RAM: 2 ГБ
- SSD: 20 ГБ
```
2. Установлен Docker Engine.
```
sudo apt install docker.io
```
3. Добавляем пользователя в группу docker
Чтобы каждый раз при работе с docker не писать "sudo" добавим нашего пользователя в группу docker
```
sudo usermod -a -G docker hw2
```
4. Создаем каталог /var/lib/postgres для хранения данных.
```
hw2@hw2:~$ sudo mkdir /var/lib/postgres
```

5.  Разворачиваем контейнер с PostgreSQL 17, смонтировав в него /var/lib/postgres.
Создаем сеть для docker
```
hw2@hw2:~$ docker network create pg-net
3db1dec0b9458d112a131bac9f925908273174d2347fee3f872121ecf672db99
```

```
hw2@hw2:~$ docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -v /host/postgres-data:/var/lib/postgresql/data postgres:17

Unable to find image 'postgres:17' locally
17: Pulling from library/postgres
8c7716127147: Pull complete
de7816569160: Pull complete
1ed9605b075e: Pull complete
ba692dbf22b7: Pull complete
d5d1559014a5: Pull complete
f6a82cec6433: Pull complete
ce1f028def8f: Pull complete
be8c3c303f61: Pull complete
2a9905500c81: Downloading  92.57MB/114.1MB
95861f01fbc3: Download complete
9138844e74a6: Download complete
29da297fd253: Download complete
4664e556e90a: Download complete
8d6e0d53c584: Download complete
```
Проверяем созданный контейнер
```
hw2@hw2:~$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
ff4e548f23a4   postgres:17   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   0.0.0.0:5432->5432/tcp   pg-server
```

6.  Разворачиваем контейнер с клиентом PostgreSQL.
```
hw2@hw2:~$ docker run -it --rm --network pg-net --name pg-client postgres:17 psql -h pg-server -p 5432 -U postgres
Password for user postgres:
psql (17.6 (Debian 17.6-2.pgdg13+1))
Type "help" for help.
postgres=#
```

7.Подключаемся из контейнера с клиентом к контейнеру с сервером и создаем таблицу с данными о перевозках.
```
postgres=# create table shipments(id serial, product_name text, quantity int, destination text);
CREATE TABLE
postgres=# insert into shipments(product_name, quantity, destination) values('bananas', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('bananas', 1500, 'Asia');
insert into shipments(product_name, quantity, destination) values('bananas', 2000, 'Africa');
insert into shipments(product_name, quantity, destination) values('coffee', 500, 'USA');
insert into shipments(product_name, quantity, destination) values('coffee', 700, 'Canada');
insert into shipments(product_name, quantity, destination) values('coffee', 300, 'Japan');
insert into shipments(product_name, quantity, destination) values('sugar', 1000, 'Europe');
insert into shipments(product_name, quantity, destination) values('sugar', 800, 'Asia');
insert into shipments(product_name, quantity, destination) values('sugar', 600, 'Africa');
insert into shipments(product_name, quantity, destination) values('sugar', 400, 'USA');
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
postgres=#
```
8. Подключаемся к контейнеру с сервером, удаляем контейнер с сервером
```
hw2@hw2:~$ docker container rm -f ff4e548f23a453827ff5629e0cd5286449fdd984240ed948b79289023e31f657
ff4e548f23a453827ff5629e0cd5286449fdd984240ed948b79289023e31f657
```
9. Создаем контейнер с сервером заново
```
hw2@hw2:~$ docker run -d --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -p 5433:5432 -v /host/postgres-data:/var/lib/postgresql/data postgres:17
17db5563fcbeaadff5b1f543c7a39d5bd375473f5f791b9a4b29dbf3b19871b8
```

10. Проверяем, что данные остались на месте.
```
hw2@hw2:~$ docker run -it --rm --network pg-net --name pg-client postgres:17 psql -h pg-server -p 5432 -U postgres
Password for user postgres:
psql (17.6 (Debian 17.6-2.pgdg13+1))
Type "help" for help.

postgres=# select * from shipments;
 id | product_name | quantity | destination
----+--------------+----------+-------------
  1 | bananas      |     1000 | Europe
  2 | bananas      |     1500 | Asia
  3 | bananas      |     2000 | Africa
  4 | coffee       |      500 | USA
  5 | coffee       |      700 | Canada
  6 | coffee       |      300 | Japan
  7 | sugar        |     1000 | Europe
  8 | sugar        |      800 | Asia
  9 | sugar        |      600 | Africa
 10 | sugar        |      400 | USA
(10 rows)

postgres=#
```
Вывод: Мы научились устанавливать докер, создавать виртуальные сети докера, запускать postgres в контейнерах и осуществлять сетевое взаимодействие между сервером и клиентом в виртуальной сети. А также освоили технологию проброса папки из хоста в контейнер volumes
