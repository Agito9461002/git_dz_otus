**ДЗ 1. SQL и реляционные СУБД. Введение в PostgreSQL**

**Предварительные действия:**
Создан новый проект на cloud.yandex.ru. Выбран Ubuntu 22.04 с дефолтными настройками. Создан ssh  ключ. Установлен PostgreSQL 14 версии. Открыты 2 вкладки в терминале.

**Действия в терминале:**
*Прим.: 1. Если не указано иное, действие совершается в первой вкладке. 2.Quote – это примечание или вопрос-ответ из ДЗ.*

ssh  vboxuser@51.250.96.142

ssh  vboxuser@51.250.96.142

	(во второй вкладке)

sudo -u postgres psql -p 5432

sudo -u postgres psql -p 5432

	(во второй вкладке)

create database iso;

\c iso;

\c  iso;

	(во второй вкладке)

\echo :AUTOCOMMIT

\set AUTOCOMMIT OFF

BEGIN;

create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;

show transaction isolation level;

	read committed

BEGIN;

	Интересно, что т.к. автокоммит отключен, то если вместо BEGIN  написать любой запрос, 	SELECT *_ FROM PERSONS; (т.е. совершить действие), то транзакция тоже открывается (iso=*#) и не закрывается.

BEGIN;

	(во второй вкладке)

insert into persons(first_name, second_name) values('sergey', 'sergeev');

select * from persons;

	(во второй вкладке)

	Видите ли вы новую запись и если да то почему?

	Ответ: Нет. В PostgreSQL на уровне изоляции Read Committed нельзя увидеть незафиксированных 	данных или изменений, внесённых в процессе выполнения запроса параллельными транзакциями. Показываются только данные уже зафиксированные до начала запроса.

commit;

select * from  persons;

	(во второй вкладке)

	Видите ли вы новую запись и если да то почему?

	Ответ: Да. Уровень изоляции Read Committed уровень изоляции позволяет увидеть изменения, внесённые успешно завершёнными транзакциями в оставшихся параллельно открытых транзакциях.

commit;

	(во второй вкладке)

set transaction isolation level repeatable read;

set transaction isolation level repeatable read;

	(во второй вкладке)

BEGIN;

BEGIN;

	(во второй вкладке)

insert into persons(first_name, second_name) values('sveta', 'svetova');

select * from persons;

	(во второй вкладке)

	Видите ли вы новую запись и если да то почему?

	Ответ: Нет. В Repeatable Read видны только те данные, которые были зафиксированы до начала транзакции.

commit;

select * from persons;

	(во второй вкладке)

	Видите ли вы новую запись и если да то почему?

	Ответ: Да. Запрос в транзакции данного уровня изоляции видит снимок данных на момент начала первого оператора в транзакци Т.е. последовательные команды SELECT в одной транзакции видят одни и те же данные; они не видят изменений, внесённых и зафиксированных другими транзакциями после начала их текущей транзакции.

commit;

	(во второй вкладке)

select * from persons;

	(во второй вкладке)

	Видите ли вы новую запись и если да то почему?

	Ответ: Да.

	Если мы, открыв 2 транзакции, добавим вначале в первой новую строку, например, ('kim', 'faleev');, сделаем commit в первой транзакции, а затем во второй добавим другую новую строку, например. ('fedor', 'fedorov'). То запрос select * from persons во второй транзакции покажет только 'fedor', 'fedorov', но по id  строк будет видно, что «что-то» пропущено. И только после commit; и второй транзакции мы увидим таблицу полностью.

[screenshot](https://cloud.mail.ru/public/8MPU/oGBMSTWLE)