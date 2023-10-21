
**Создайте новый кластер PostgresSQL 14**<br>
Для выполнения данной домашней работы я буду использовать уже уимеющийся кластер PostgreSQL 15

**Зайдите в созданный кластер под пользователем postgres**
```
sudo -su postgres psql
```

**Создайте новую базу данных testdb**
```
CREATE DATABASE testdb;
```

**Зайдите в созданную базу данных под пользователем postgres**
```
\c testdb
```

**Создайте новую схему testnm**
```
create schema testnm;
```

**Создайте новую таблицу t1 с одной колонкой c1 типа integer**
```
create table t1(c1 int);
```

**Вставьте строку со значением c1=1**
```
insert into t1 (c1) values (1);
```

**Создайте новую роль readonly**
```
create role readonly;
```

**Дайте новой роли право на подключение к базе данных testdb**
```
grant connect on database testdb to readonly;
```

**Дайте новой роли право на использование схемы testnm**
```
grant usage on schema testnm to readonly;
```

**Дайте новой роли право на select для всех таблиц схемы testnm**
```
grant select on all tables in schema testnm to readonly;
```

**Создайте пользователя testread с паролем test123**
```
create user testread with password 'test123';
```

**Дайте роль readonly пользователю testread**
```
grant readonly to testread;
```

**Зайдите под пользователем testread в базу данных testdb**
```
сделайте select * from t1;
```
получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)<br>
не получилось<br>
напишите что именно произошло в тексте домашнего задания<br>
```
testdb=> select * from t1;
ERROR:  permission denied for table t1
```
у вас есть идеи почему? ведь права то дали?<br>
при создании таблицы t1 не указали схему, следовательно таблица создалась в схеме public. При этом мы давали право на select в схеме testnm<br>
вернитесь в базу данных testdb под пользователем postgres<br>
удалите таблицу t1
```
drop table t1;
```
Cоздайте ее заново но уже с явным указанием имени схемы testnm
```
create table testnm.t1(c1 int);
```
вставьте строку со значением c1=1
```
insert into testnm.t1 (c1) values (1);
```
зайдите под пользователем testread в базу данных testdb
```
сделайте select * from testnm.t1;
```
получилось?
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
Не получилось<br>
есть идеи почему? если нет - смотрите шпаргалку<br>
На сколько я понял, что выданные права не наследуются на таблицы, которые были созданы после того, как права были выданы. Нужно снова выдать право select на таблицу t1<br>
как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку<br>
Каждый раз после создания новой схемы выдавать права пользователю<br>
сделайте 
```
select * from testnm.t1;
```
получилось?<br>
ура!
```
grant select on all tables in schema testnm to readonly;
```
Получилось после повторной выдачи прав.<br>

**Теперь попробуйте выполнить команду**
```
create table t2(c1 integer); insert into t2 values (2);
```
Не получилось, т.к. у меня используется кластер 15 версии. В 15 версии изменили права в схеме public.<br>
А как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?<br>
в PostgreSQL 14 в схеме public по умолчанию есть право CREATE для всех.<br>
**Есть идеи как убрать эти права? если нет - смотрите шпаргалку**
```
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```
**Теперь попробуйте выполнить команду**
```
create table t3(c1 integer); insert into t2 values (2);
```
Расскажите что получилось и почему<br>
После того, как отобрали право CREATE новые таблицы в схеме public создать не получится (для не владельцев схемы)

