
## создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере , например postgres2022-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально на уровне GCP)
```bash
сделал по инструкции Yandex_Cloud-25239-8dd990.md

Идентификатор
b1gvglfdulq11k3cflgm
Статус
Active
Имя
cloud-konstantintolmachov
Дата создания
24 июля 2022, в 12:44
Организация
organization-konstantintolmachov

```

## далее создать инстанс виртуальной машины Compute Engine с дефолтными параметрами
```bash

Обзор
Идентификатор
fhm2rip87flrku7ji1ck
Статус
Stopped
Имя
otus-db-pg-vm-1
Дата создания
24 июля 2022, в 22:10
Внутренний FQDN
otus-db-pg-vm-1.ru-central1.internal
Зона доступности
ru-central1-a
Ресурсы
Платформа
Intel Ice Lake
Гарантированная доля vCPU
100%
vCPU
2
RAM
4 ГБ
Объём дискового пространства
15 ГБ

```

## добавить свой ssh ключ в GCE metadata
```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQD... olimp@Olimp    
добавил при сразу при создание ВМ
```

## зайти удаленным ssh (первая сессия), не забывайте про ssh-add
```bash
olimp@Olimp:~/.ssh$

-- запускаем агента, если не запущен
eval `ssh-agent -s`
ssh-add /home/olimp/.ssh/id_rsa
Enter passphrase for /home/olimp/.ssh/id_rsa:
Identity added: /home/olimp/.ssh/id_rsa (olimp@Olimp)

olimp@Olimp:~/.ssh$ ssh otus@51.250.7.75
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:ikoGhx7KQJFDkE/zU9pml0umhoSM1BjKmCr2rhgdo7E.
Please contact your system administrator.
Add correct host key in /home/olimp/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/olimp/.ssh/known_hosts:2
  remove with:
  ssh-keygen -f "/home/olimp/.ssh/known_hosts" -R "51.250.7.75"
Host key for 51.250.7.75 has changed and you have requested strict checking.
Host key verification failed.
olimp@Olimp:~/.ssh$ ssh otus@51.250.13.197
The authenticity of host '51.250.13.197 (51.250.13.197)' can't be established.
ED25519 key fingerprint is SHA256:sBXUI876ghRP0urd3QWhFQtfROxFfOWdUzsy4AiOdg8.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '51.250.13.197' (ED25519) to the list of known hosts.
Enter passphrase for key '/home/olimp/.ssh/id_rsa':
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Sun Jul 24 19:12:07 2022 from 178.66.109.168
otus@otus-db-pg-vm-1:~$

```

## поставить PostgreSQL
```bash

otus@otus-db-pg-vm-1:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
otus@otus-db-pg-vm-1:~$

```

## зайти вторым ssh (вторая сессия)
```bash
olimp@Olimp:/mnt/c/Windows/System32$ ssh otus@51.250.13.197
Enter passphrase for key '/home/olimp/.ssh/id_rsa':
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Sun Jul 24 20:26:05 2022 from 178.66.109.168
otus@otus-db-pg-vm-1:~$

```

## запустить везде psql из под пользователя postgres
```bash
otus@otus-db-pg-vm-1:~$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#
```

## выключить auto commit
```bash
postgres=# \echo :AUTOCOMMIT
on
postgres=# \set AUTOCOMMIT OFF
postgres=#
```

## сделать в первой сессии новую таблицу и наполнить ее данными create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
```bash

postgres=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
CREATE TABLE
INSERT 0 1
INSERT 0 1
COMMIT
postgres=#
```

## посмотреть текущий уровень изоляции: show transaction isolation level
```bash
postgres=# show transaction isolation level
postgres-# ;
 transaction_isolation
-----------------------
 read committed
(1 row)

```

## начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
```bash
postgres=*# BEGIN;
WARNING:  there is already a transaction in progress
BEGIN
postgres=*# commit;
COMMIT
postgres=# BEGIN;
BEGIN
postgres=*#

```

## в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sergey', 'sergeev');
```bash
postgres=*# into persons(first_name, second_name) values('sergey', 'sergeev');
ERROR:  syntax error at or near "into"
LINE 1: into persons(first_name, second_name) values('sergey', 'serg...
        ^
postgres=!# insert into persons(first_name, second_name) values('sergey', 'sergeev');
ERROR:  current transaction is aborted, commands ignored until end of transaction block
postgres=!# BEGIN;
ERROR:  current transaction is aborted, commands ignored until end of transaction block
postgres=!# commit;
ROLLBACK
postgres=# BEGIN;
BEGIN
postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
INSERT 0 1
postgres=*#
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)

```

## сделать select * from persons во второй сессии
```bash
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)

```

## видите ли вы новую запись и если да то почему?
```bash
нет
```

## завершить первую транзакцию - commit;
```bash
postgres=*# commit;
COMMIT
```

## сделать select * from persons во второй сессии
```bash
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```

## видите ли вы новую запись и если да то почему?
```bash
да, потому что уровень изоляции по умолчанию read committed, и возможна коллизия 
- не повторяющееся чтение
```

## завершите транзакцию во второй сессии
```bash
postgres=*# commit;
COMMIT
```

## начать новые но уже repeatable read транзации - set transaction isolation level repeatable read;
```bash
postgres=# set transaction isolation level repeatable read;
SET
postgres=*# begin;
WARNING:  there is already a transaction in progress
BEGIN
postgres=*#
```

## в первой сессии добавить новую запись insert into persons(first_name, second_name) values('sveta', 'svetova');
```bash
postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
postgres=*#
```

## сделать select * from persons во второй сессии
```bash
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```

## видите ли вы новую запись и если да то почему?
```bash
нет
```

## завершить первую транзакцию - commit;
```bash
postgres=*# commit;
COMMIT
```

## сделать select * from persons во второй сессии
```bash
postgres=*# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
```

## видите ли вы новую запись и если да то почему?
```bash
нет, потому что уровень изоляции repeatable read и не возможна коллизия - не повторяющееся чтение
```

## завершить вторую транзакцию
```bash
postgres=*# commit;
COMMIT
```

## сделать select * from persons во второй сессии
```bash
postgres=# select * from persons;
 id | first_name | second_name
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
  5 | sveta      | svetova
(4 rows)
```

## видите ли вы новую запись и если да то почему?
```bash
да, потому что транзакции в первой сессии была завершена комитом
и запрос во второй сессии уже видит все записи в новой транзакции
```
