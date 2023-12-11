Из-за ограниченых ресурсов домашнее задание будет выполнено на одной ВМ.<br>
Создаем дополнительно 2 кластера:
```
pg_createcluster 15 main2
pg_createcluster 15 main3
```
**На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.**
```
create database repl;
\c repl
create table test as select generate_series(1,100) as id, md5(random()::text)::char(10) as name;
create table test2 (id serial, name char(10));
```

**Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.**
```
ALTER SYSTEM SET wal_level = logical
рестарт кластера
create publication test1 for table test;
```

```
CREATE SUBSCRIPTION test2
CONNECTION 'host=localhost port=5433 user=postgres password=postgres dbname=repl'
PUBLICATION test2 WITH (copy_data = true);
```

**На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.**
```
create database repl;
\c repl
create table test (id serial, name char(10));
create table test2 as select generate_series(1,100) as id, md5(random()::text)::char(10) as name;
```

**Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.**
```
create publication test2 for table test2;
```

```
CREATE SUBSCRIPTION test1
CONNECTION 'host=localhost port=5432 user=postgres password=postgres dbname=repl'
PUBLICATION test1 WITH (copy_data = true);
```


**3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).**
```
create table test (id serial, name char(10));
create table test2 (id serial, name char(10));
```

```
CREATE SUBSCRIPTION test3_1
CONNECTION 'host=localhost port=5432 user=postgres password=postgres dbname=repl'
PUBLICATION test1 WITH (copy_data = true);

CREATE SUBSCRIPTION test3_2
CONNECTION 'host=localhost port=5433 user=postgres password=postgres dbname=repl'
PUBLICATION test2 WITH (copy_data = true);
```

**Задание со звездочкой. Реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.**
Создаем четвертый кластер:
```
pg_createcluster 15 main4
```
Удаляем все созданные файлы для четвертого кластера:
```
rm -rf /var/lib/postgresql/15/main4
```

Делаем бэкап третьего кластера в четвертый кластер с ключами D и R:
```
sudo -i -u postgres
pg_basebackup -p 5434 -R -D /var/lib/postgresql/15/main4
echo 'hot_standby = on' >> /var/lib/postgresql/15/main4/postgresql.auto.conf
```
Запускаем четвертый кластер:
```
pg_ctlcluster 15 main5 start
```


