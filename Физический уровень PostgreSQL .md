
## создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a
```bash

Идентификатор
fhm0uo2knv4e112fpknq
Статус
Running
Имя
otus-db-pg-vm-1
Дата создания
31 июля 2022, в 19:59
Внутренний FQDN
otus-db-pg-vm-1.ru-central1.internal
Зона доступности
ru-central1-a

```

## поставьте на нее PostgreSQL 14 через sudo apt 
```bash
sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-14

done
```

## проверьте что кластер запущен через sudo -u postgres pg_lsclusters
```bash
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
otus@otus-db-pg-vm-1:~$
```

## зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
```bash
sudo -u postgres psql
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('123');
INSERT 0 1
postgres=#
\q
```

## остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
```bash
otus@otus-db-pg-vm-1:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@14-main
otus@otus-db-pg-vm-1:~$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
otus@otus-db-pg-vm-1:~$
```

## создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
![подключенный к vm диск](.\images\6\новый_диск.jpg)

## добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
![подключенный к vm диск](.\images\6\подключенный_к_vm_диск.jpg)

[каталог с изображениями к дз](https://github.com/K-Otus2022/postgre-dba-2022-06/tree/3e51ff11667f29ed5fec6f4a9ce254c4a8cdf589/images/6)



## проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - 
## https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
```bash
otus@otus-db-pg-vm-1:~$ sudo apt install parted
Reading package lists... Done
Building dependency tree
Reading state information... Done
parted is already the newest version (3.3-4ubuntu0.20.04.1).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
otus@otus-db-pg-vm-1:~$ sudo parted -l | grep Error
Error: /dev/vdb: unrecognised disk label
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo parted /dev/vdb mklabel gpt
Information: You may need to update /etc/fstab.

otus@otus-db-pg-vm-1:~$ sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
Information: You may need to update /etc/fstab.

otus@otus-db-pg-vm-1:~$ pvresize /dev/vdb

Command 'pvresize' not found, but can be installed with:

apt install lvm2
Please ask your administrator.

otus@otus-db-pg-vm-1:~$ sudo apt install lvm2
otus@otus-db-pg-vm-1:~$ sudo pvresize /dev/vdb
  Failed to find device for physical volume "/dev/vdb".
  0 physical volume(s) resized or updated / 0 physical volume(s) not resized
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    252:0    0  15G  0 disk
├─vda1 252:1    0   1M  0 part
└─vda2 252:2    0  15G  0 part /
vdb    252:16   0  10G  0 disk
└─vdb1 252:17   0  10G  0 part
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo mkfs.ext4 -L datapartition /dev/vdb1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 2620928 4k blocks and 655360 inodes
Filesystem UUID: c53b02b8-08bd-47fa-86d8-acc9eef23ae2
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

otus@otus-db-pg-vm-1:~$
otus@otus-db-pg-vm-1:~$ sudo lsblk --fs
NAME   FSTYPE LABEL         UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda
├─vda1
└─vda2 ext4                 82afb880-9c95-44d6-8df9-84129f3f2cd1   11.5G    18% /
vdb
└─vdb1 ext4   datapartition c53b02b8-08bd-47fa-86d8-acc9eef23ae2
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo mount -o defaults /dev/vdb1 /mnt/data
otus@otus-db-pg-vm-1:~$
otus@otus-db-pg-vm-1:~$ df -h -x tmpfs
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
/dev/vda2        15G  2.7G   12G  19% /
/dev/vdb1       9.8G   24K  9.3G   1% /mnt/data
otus@otus-db-pg-vm-1:~$

```

## сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
```bash
otus@otus-db-pg-vm-1:~$ sudo chown -R postgres:postgres /mnt/data/
otus@otus-db-pg-vm-1:~$
otus@otus-db-pg-vm-1:~$ echo "success" | sudo tee /mnt/data/test_file
success
otus@otus-db-pg-vm-1:~$ cat /mnt/data/test_file
success
otus@otus-db-pg-vm-1:~$ sudo rm /mnt/data/test_file
otus@otus-db-pg-vm-1:~$

```

## перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
```bash
otus@otus-db-pg-vm-1:~$ mv /var/lib/postgresql/14 /mnt/data
mv: cannot create directory '/mnt/data/14': Permission denied
otus@otus-db-pg-vm-1:~$ sudo mv /var/lib/postgresql/14 /mnt/data
otus@otus-db-pg-vm-1:~$
```

## попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
```bash
otus@otus-db-pg-vm-1:~$ sudo -u postgres pg_ctlcluster 14 main start
Error: /var/lib/postgresql/14/main is not accessible or does not exist
otus@otus-db-pg-vm-1:~$
```

## напишите получилось или нет и почему
```bash
нет, потому что каталог по умолчанию /var/lib/postgresql/14/main с файлами базы был перенесен на другой диск

```

## задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
```bash
otus@otus-db-pg-vm-1:~$ sudo nano /etc/postgresql/14/main/postgresql.conf
otus@otus-db-pg-vm-1:~$
```

## апишите что и почему поменяли
```bash
в разделе FILE LOCATIONS значение для data_directory
data_directory = '/var/lib/postgresql/14/main'          # use data in another directory
                                        # (change requires restart)

на data_directory = '/mnt/data/14/main'

```

## попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
```bash
 otus@otus-db-pg-vm-1:/mnt/data/14$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
Removed stale pid file.

 otus@otus-db-pg-vm-1:/mnt/data/14$ sudo -u postgres pg_lsclusters
Ver Cluster Port Status Owner    Data directory    Log file
14  main    5432 online postgres /mnt/data/14/main /var/log/postgresql/postgresql-14-main.log
otus@otus-db-pg-vm-1:/mnt/data/14$

```

## напишите получилось или нет и почему
```bash
да, потому что новые изменения в конфигурационном файле были применены после рестарта кластера postgres

```

## зайдите через через psql и проверьте содержимое ранее созданной таблицы
```bash
 otus@otus-db-pg-vm-1:/mnt/data/14$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# /l
postgres-# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres-# \c postgres
You are now connected to database "postgres" as user "postgres".
postgres-# select * from test;
ERROR:  syntax error at or near "/"
LINE 1: /l
        ^
postgres=# ;
postgres=# select * from test;
 c1
-----
 123
(1 row)

```

## задание со звездочкой *: не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск ## который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали ## и что в итоге получилось.
```bash
 
вместо создания новой vm отсоединил диск у otus-db-pg-vm-1
запустил vm
otus@otus-db-pg-vm-1:~$ sudo rm  /mnt/data
rm: cannot remove '/mnt/data': Is a directory
otus@otus-db-pg-vm-1:~$ sudo rm -rf /mnt/data
otus@otus-db-pg-vm-1:~$ cd /mnt/data
-bash: cd: /mnt/data: No such file or directory
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo lsblk --fs
NAME   FSTYPE LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda
├─vda1
└─vda2 ext4         82afb880-9c95-44d6-8df9-84129f3f2cd1   11.5G    17% /

остановил vm и снова подключил внешний диск
снова запустил vm и примонтировал диск

otus@otus-db-pg-vm-1:~$ Connection to 51.250.6.239 closed by remote host.
Connection to 51.250.6.239 closed.
olimp@Olimp:~$ ssh otus@51.250.6.239
Enter passphrase for key '/home/olimp/.ssh/id_rsa':
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Sun Jul 31 18:26:56 2022 from 178.66.109.168
otus@otus-db-pg-vm-1:~$ sudo lsblk --fs
NAME   FSTYPE LABEL         UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda
├─vda1
└─vda2 ext4                 82afb880-9c95-44d6-8df9-84129f3f2cd1   11.5G    17% /
vdb
└─vdb1 ext4   datapartition c53b02b8-08bd-47fa-86d8-acc9eef23ae2
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo mount -o defaults /dev/vdb1 /mnt/data
mount: /mnt/data: mount point does not exist.
otus@otus-db-pg-vm-1:~$ sudo mkdir -p /mnt/data
otus@otus-db-pg-vm-1:~$ sudo chown -R postgres:postgres /mnt/data/
otus@otus-db-pg-vm-1:~$ sudo mount -o defaults /dev/vdb1 /mnt/data
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo -u postgres pg_ctlcluster 14 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@14-main
otus@otus-db-pg-vm-1:~$

otus@otus-db-pg-vm-1:~$ sudo -u postgres psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres=# \c postgres
You are now connected to database "postgres" as user "postgres".
postgres=# select * from test;
 c1
-----
 123
(1 row)

postgres=#

```
