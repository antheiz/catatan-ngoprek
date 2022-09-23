# Replikasi Database menggunakan Docker MySQL Images

Konfigurasi MySQL Replication Master ⭢ Slave menggunakan Docker Images

**Replikasi database adalah proses menyalin data dari suatu Database server (the master) ke suatu atau lebih Database server lainnya (the slaves).**

## Referensi

- https://github.com/wagnerjfr/mysql-master-slaves-replication-docker
- https://github.com/wagnerjfr/docker-machine-master-slave-mysql-replication

## 1. Ringkasan

Kita akan memulai dengan membuat suatu Docker network dengan nama **replicanet**, kemudian dilanjutkan 
dengan pull images docker **mysql 8.0** dari Docker Hub (https://hub.docker.com/r/mysql/mysql-server). Selanjutnya akan membuat 
2 container (1 master dan 1 slaves).

## 2. Pull MySQL Server Images

Untuk download *MySQL Community Edition Image* gunakan perintah:

```
docker pull mysql/mysql-server:tag
```

Jika `:tag` dihilangkan, maka docker akan melakukan pull *tag* versi terbaru, dan jika kita ingin pull 
untuk versi tertentu gunakan perintah:

Contoh:
```
docker pull mysql/mysql-server
docker pull mysql/mysql-server:5.7
docker pull mysql/mysql-server:8.0
```

Pada contoh kali ini, kita akan menggunakan ***mysql/mysql-server:8.0***

## 3. Membuat Docker network

Ikuti perintah berikut untuk membuat network:

```
$ docker network create replicanet
``` 

Untuk melihat semua docker network:

```
$ docker network ls
```

## 4. Membuat 2 MySQL Container

Jalankan perintah berikut ini pada terminal:

```
docker run -d --rm --name=master --net=replicanet --hostname=master \
  -v $PWD/d0:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=salupa \
  mysql/mysql-server:8.0 \
  --server-id=1 \
  --log-bin='mysql-bin-1.log'

docker run -d --rm --name=slave --net=replicanet --hostname=slave \
  -v $PWD/d1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=salupa \
  mysql/mysql-server:5.7 \
  --server-id=2
```

Jalankan perintah berikut untuk melihat status dari containers yang dijalankan:

```
$ docker ps -a
```
```zsh
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                            PORTS                 NAMES
8f8ceffd4580        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   7 seconds ago       Up 7 seconds (health: starting)   3306/tcp, 33060/tcp   master
8a10c0c92350        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   7 seconds ago       Up 5 seconds (health: starting)   3306/tcp, 33060/tcp   slave
```
Server masih dalam status ***(health:starting)***, tunggu sampai status ***(healthy)*** sebelum menjalankan perintah berikutnya.

```zsh
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                    PORTS                 NAMES
8f8ceffd4580        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   34 seconds ago      Up 34 seconds (healthy)   3306/tcp, 33060/tcp   master
8a10c0c92350        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   34 seconds ago      Up 32 seconds (healthy)   3306/tcp, 33060/tcp   slave
```
Sekarang container telah siap untuk konfigurasi replikasi database.


## 5. Konfigurasi master dan slaves

### 5.1 Master

Kita langsung mulai konfigurasi master node

*[Optional]* Jika ingin menggunakan semisynchronous replication:
```
docker exec -it master mysql -uroot -psalupa \
  -e "INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';" \
  -e "SET GLOBAL rpl_semi_sync_master_enabled = 1;" \
  -e "SET GLOBAL rpl_semi_sync_master_wait_for_slave_count = 2;" \
  -e "SHOW VARIABLES LIKE 'rpl_semi_sync%';"
```

*[Optional]* Output dari semisynchronous setup pada master:
```zsh
mysql: [Warning] Using a password on the command line interface can be insecure.
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | ON         |
| rpl_semi_sync_master_timeout              | 10000      |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 2          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
+-------------------------------------------+------------+
```

Konfigurasi *master node* dengan membuat replication user dan tampilkan status replication:
```
docker exec -it master mysql -uroot -pmypass \
  -e "DROP USER 'repl'@'%'"; \
  -e "CREATE USER 'repl'@'%' IDENTIFIED BY 'slavepass';" \
  -e "GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';" \
  -e "SHOW MASTER STATUS;"
```
Output:
```zsh
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+----------+--------------+------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+--------------------+----------+--------------+------------------+-------------------+
| mysql-bin-1.000003 |      595 |              |                  |                   |
+--------------------+----------+--------------+------------------+-------------------+
```