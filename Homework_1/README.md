Генерируем публичный и приватный ssh ключи в программе puttygen.exe и сохраняем их:
![](ssh_key_gen.jpg)


Добавляем SSH ключ на хост:

echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCDNG5J6iEIC6oZd6a4zaogJxWq8sPSig6swAlnQ81PeWF/wFa+ocwd2sBg2vCeV5T9sr3s0V5iyA+2RxTYXBD8vr8lpeI6K2uL60sI/jfIJ30Q1+AHufGeAGfTa/FkK3BhflgXTs6iAbVrUH9NOYVmSpWy/rXWHTL8zJK17K9Ndy+GixCqfapcaM6+HQLt+nph8Pqmp+RUwkE2Mm0PvcTwglMV2WVF45MBmyVvRy4xy52mGrDI2eQ/B9F2UQp2WYupAjaeTiAPC1PyFNLzOxX3nVERQBLXEZzHq3tJZtFUeaBCWauh0r0yYuzvB1qQd9MLsvBiKJdvqNj3SulhkeOV rsa-key-20231008" >> .ssh/authorized_keys

Устанавливаем PostgreSQL 15:

sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql-15

Отключаем auto commit:
\set AUTOCOMMIT OFF

в первой сессии новую таблицу и наполнить ее данными:
create table persons(id serial, first_name text, second_name text);
insert into persons(first_name, second_name) values('ivan', 'ivanov');
insert into persons(first_name, second_name) values('petr', 'petrov');
commit;

посмотреть текущий уровень изоляции: 
show transaction isolation level;

картинка

в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
сделать select * from persons; во второй сессии
Новая запись не видна, т.к. в первой сессии транзакция не закоммичена

сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
Да, т.к. транзакция в первой сессии завершена и в режиме read commited видны изменения, которые были внескены в других сессиях.

начать новые но уже repeatable read транзации
set transaction isolation level repeatable read;

в первой сессии добавить новую запись
insert into persons(first_name, second_name) values('sveta', 'svetova');

сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
Нет, т.к. транзакция не была завершена в первой сессии

завершить первую транзакцию
commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему?
Нет, т.к. в режиме repeatable read видны только те изменения, которые были сделаны до начала транзакции. При выполнении запроса select * from persons создается новая транзакция и пока она не будет завершена, изменения, внесенные в других сессиях\транзацкиях в данной транзакции видны не будут.

завершить вторую транзакцию
commit;
сделать select * from persons во второй сессии
видите ли вы новую запись и если да то почему
Да, т.к. после завершения транзакции в данной сессии стали видны изменния, которые были сделаны во время выполнения данной транзакции. (особенность уровня изоляции транзакций repeatable read)
