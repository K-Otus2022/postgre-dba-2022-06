
## сделать в GCE инстанс с Ubuntu 20.04 или развернуть докер любым удобным способом
```bash
сделал по инструкции Yandex_Cloud-25239-8dd990.md
+ https://cloud.yandex.ru/docs/container-registry/tutorials/run-docker-on-vm

Идентификатор
fhm4b7ad0m9pe6m3ga1q
Статус
Running
Имя
otus-docker-vm-1
Дата создания
31 июля 2022, в 16:28

olimp@Olimp:~$ ssh otus@51.250.8.64
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)

otus@otus-docker-vm-1:~$ docker

Command 'docker' not found, but can be installed with:

apt install docker.io
Please ask your administrator.

```

## поставить на нем Docker Engine
```bash

otus@otus-docker-vm-1:~$ sudo apt install docker.io
Reading package lists... Done
--

otus@otus-docker-vm-1:~$ docker -v
Docker version 20.10.12, build 20.10.12-0ubuntu2~20.04.1

```

## сделать каталог /var/lib/postgres
```bash
otus@otus-docker-vm-1:~$ mkdir /var/lib/postgres
mkdir: cannot create directory ‘/var/lib/postgres’: Permission denied
otus@otus-docker-vm-1:~$ sudo mkdir /var/lib/postgres
```

## развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
```bash
otus@otus-docker-vm-1:~$ sudo docker network create pg-net
6e2a8298c1fa3ac317ba111d7f12cd3b02f26f0689df0e1a1ed3f9f523979794

otus@otus-docker-vm-1:~$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
Unable to find image 'postgres:14' locally
14: Pulling from library/postgres
461246efe0a7: Pull complete
8d6943e62c54: Pull complete
558c55f04e35: Pull complete
186be55594a7: Pull complete
f38240981157: Pull complete
e0699dc58a92: Pull complete
066f440c89a6: Pull complete
ce20e6e2a202: Pull complete
c0f13eb40c44: Pull complete
3d7e9b569f81: Pull complete
2ab91678d745: Pull complete
ffc80af02e8a: Pull complete
f3a57056b036: Pull complete
Digest: sha256:3e2eba0a6efbeb396e086c332c5a85be06997d2cf573d34794764625f405df4e
Status: Downloaded newer image for postgres:14
b355ba9a4bd447e1c5852ad322d316aadbb6ac0f4b0204d65ca64375d0f8336e
otus@otus-docker-vm-1:~$ sudo docker ps -ls
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES       SIZE
b355ba9a4bd4   postgres:14   "docker-entrypoint.s…"   22 seconds ago   Up 20 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker   63B (virtual 376MB)
```

## развернуть контейнер с клиентом postgres
```bash
otus@otus-docker-vm-1:~$ sudo docker run -it --network pg-net --name pg-client postgres:14 /bin/bash
root@23f9e4206f2f:/#

```

## подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
```bash
sudo -u postgres psql -h pg-docker -U postgres

docker run -it pg-client psql -h pg-docker -U postgres

root@23f9e4206f2f:/# psql -h pg-docker -U postgres
Password for user postgres:
psql: error: connection to server at "pg-docker" (172.18.0.2), port 5432 failed: FATAL:  password authentication failed for user "postgres"
root@23f9e4206f2f:/# su postgres
postgres@23f9e4206f2f:/$ psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
postgres=# CREATE DATABASE iso;
CREATE DATABASE
postgres=# \c iso
You are now connected to database "iso" as user "postgres".
iso=# CREATE TABLE test (i serial, amount int);
NSERT INTO test(amount) VACREATE TABLE
iso=# INSERT INTO test(amount) VALUES (100);
INSERT 0 1
iso=# INSERT INTO test(amount) VALUES (500);
INSERT 0 1
iso=# SELECT * FROM test;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)

iso=#
```

## подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/места установки докера
```bash
olimp@Olimp:/mnt/c/Windows/System32$ psql -p 5432 -U postgres -h 51.250.8.64 -d postgres -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg22.04+1))
Type "help" for help.

postgres=#

postgres-# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 iso       | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)


```

## удалить контейнер с сервером 
```bash
iso=# \q
postgres@23f9e4206f2f:/$ exit
exit
root@23f9e4206f2f:/# exit
exit
otus@otus-docker-vm-1:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                      PORTS                                       NAMES
23f9e4206f2f   postgres:14   "docker-entrypoint.s…"   14 minutes ago   Exited (0) 17 seconds ago                                               pg-client
b355ba9a4bd4   postgres:14   "docker-entrypoint.s…"   42 minutes ago   Up 42 minutes               0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
otus@otus-docker-vm-1:~$ sudo docker rm b35
Error response from daemon: You cannot remove a running container b355ba9a4bd447e1c5852ad322d316aadbb6ac0f4b0204d65ca64375d0f8336e. Stop the container before attempting removal or force remove
otus@otus-docker-vm-1:~$ sudo docker stop b35
b35
otus@otus-docker-vm-1:~$ sudo docker rm b35
b35
otus@otus-docker-vm-1:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                          PORTS     NAMES
23f9e4206f2f   postgres:14   "docker-entrypoint.s…"   15 minutes ago   Exited (0) About a minute ago             pg-client
otus@otus-docker-vm-1:~$


```

## создать его заново 
```bash
otus@otus-docker-vm-1:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                          PORTS                                       NAMES
92f831aacf1e   postgres:14   "docker-entrypoint.s…"   3 seconds ago    Up 2 seconds                    0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
23f9e4206f2f   postgres:14   "docker-entrypoint.s…"   16 minutes ago   Exited (0) About a minute ago                                               pg-client
otus@otus-docker-vm-1:~$
```

## подключится снова из контейнера с клиентом к контейнеру с сервером 
```bash
otus@otus-docker-vm-1:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                          PORTS                                       NAMES
92f831aacf1e   postgres:14   "docker-entrypoint.s…"   3 seconds ago    Up 2 seconds                    0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
23f9e4206f2f   postgres:14   "docker-entrypoint.s…"   16 minutes ago   Exited (0) About a minute ago                                               pg-client
otus@otus-docker-vm-1:~$ sudo docker rm 23
23
otus@otus-docker-vm-1:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```

## проверить, что данные остались на месте 
```bash
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 iso       | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# \c iso
You are now connected to database "iso" as user "postgres".
iso=# select * from test
iso-# ;
 i | amount
---+--------
 1 |    100
 2 |    500
(2 rows)

iso=#
```
