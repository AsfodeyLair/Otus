- Выключить auto commit: 

> \set AUTOCOMMIT off \echo :AUTOCOMMIT

Cделать в первой сессии новую таблицу и наполнить ее данными: 

> create table persons(id serial, first_name text, second_name text);
> insert into persons(first_name, second_name) values('ivan','ivanov');  
> insert into persons(first_name, second_name) values('petr','petrov');  
> commit;

Посмотреть текущий уровень изоляции: 

> show transaction isolation level; 
> transaction_isolation
> read committed

Начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции. В первой сессии добавить новую запись 

> insert into persons(first_name, second_name) values('sergey', 'sergeev');

Сделать во второй сессии: 

> select * from persons; 
> id | first_name | second_name 
> 1 | ivan       |ivanov 
> 2 | petr       | petrov


Видите ли вы новую запись и если да то почему?

    Не видим потому что: 
    1. Выключили auto commit 
    2. Не выполнили явный commit для транзакции с Insert 
    3. Выполнили транзакцию SELECT с уровнем изоляции по умолчанию (read committed)

Завершить первую транзакцию - 

> commit;

Сделать во второй сессии.

> select * from persons
> id	 |	first_name | second_name
> 5	|	ivan      |	ivanov
> 6	|	petr       | petrov
> 8	|	sergey     |	sergeev

 Видите ли вы новую запись и если да то почему?

    Видим, потому что сделали ручной commit.

Завершите транзакцию во второй сессии.

Начать новые но уже repeatable read транзакции: 
Устанавливаем уровень изоляции транзакций repeatable read: 

> set transaction isolation level repeatable read;

В первой сессии добавить новую запись: 
> insert into persons(first_name, second_name) values('sveta', 'svetova');

Сделать во второй сессии: 
> select * from persons;

Видите ли вы новую запись и если да то почему? 

    Нет, потому что не включали AUTOCOMMIT

Завершить первую транзакцию 

> commit;

Сделать во второй сессии 
> select * from persons
> id	 |	first_name | second_name
> 5	|	ivan      |	ivanov
> 6	|	petr       | petrov
> 8	|	sergey     |	sergeev

Видите ли вы новую запись и если да то почему? 

    Нет, потому что установили уровень изоляции repeatable read в текущей транзакции.

Завершить вторую транзакцию, сделать во второй сессии: 
> select * from persons
> id | first_name | second_name
> 5 | ivan       | ivanov
> 6 | petr       | petrov
> 8 | sergey     | sergeev
> 9 | sveta      | svetova

Видите ли вы новую запись и если да то почему? 

    Да потому что начали новую транзакцию со стандартным уровнем изоляции - read commited.
