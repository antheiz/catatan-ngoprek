# Replikasi Database menggunakan Docker MySQL Images

Konfigurasi MySQL Replication Master ⭢ Slave menggunakan Docker Images

**Replikasi database adalah proses menyalin data dari suatu Database server (the master) ke suatu atau lebih Database server lainnya (the slave).**

## 1. Ringkasan

Kita akan memulai dengan membuat suatu Docker network dengan nama **replicanet**, kemudian dilanjutkan 
dengan pull images docker **mysql 8.0** dari Docker Hub (https://hub.docker.com/r/mysql/mysql-server). Selanjutnya akan membuat 
2 container (1 master dan 1 slaves).

Sebelum memulai pastikan telah menginstall docker pada Device anda,
Untuk mengecek apakah docker sudah terinstall gunakan perintah:

```
$ docker --version
```
Output:
```console
Docker version 20.10.17, build 100c701
```

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
$ docker run -d --rm --name=master --net=replicanet --hostname=master \
  -v $PWD/d0:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=salupa \
  mysql/mysql-server:8.0 \
  --server-id=1 \
  --log-bin='mysql-bin-1.log'

$ docker run -d --rm --name=slave --net=replicanet --hostname=slave \
  -v $PWD/d1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=salupa \
  mysql/mysql-server:5.7 \
  --server-id=2
```

Jalankan perintah berikut untuk melihat status dari containers yang dijalankan:

```
$ docker ps -a
```
```console
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                            PORTS                 NAMES
8f8ceffd4580        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   7 seconds ago       Up 7 seconds (health: starting)   3306/tcp, 33060/tcp   master
8a10c0c92350        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   7 seconds ago       Up 5 seconds (health: starting)   3306/tcp, 33060/tcp   slave
```
Server masih dalam status ***(health:starting)***, tunggu sampai status ***(healthy)*** sebelum menjalankan perintah berikutnya.

```console
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS                    PORTS                 NAMES
8f8ceffd4580        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   34 seconds ago      Up 34 seconds (healthy)   3306/tcp, 33060/tcp   master
8a10c0c92350        mysql/mysql-server:5.7   "/entrypoint.sh --se…"   34 seconds ago      Up 32 seconds (healthy)   3306/tcp, 33060/tcp   slave
```
Sekarang container telah siap untuk konfigurasi replikasi database.


## 5. Konfigurasi master dan slave

### 5.1 Master

Kita langsung mulai konfigurasi **master node**

*[Optional]* Jika ingin menggunakan semisynchronous replication:
```
$ docker exec -it master mysql -uroot -psalupa \
  -e "INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';" \
  -e "SET GLOBAL rpl_semi_sync_master_enabled = 1;" \
  -e "SET GLOBAL rpl_semi_sync_master_wait_for_slave_count = 2;" \
  -e "SHOW VARIABLES LIKE 'rpl_semi_sync%';"
```

*[Optional]* Output dari semisynchronous setup pada master:
```console
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
$ docker exec -it master mysql -uroot -psalupa \
  -e "DROP USER 'repl'@'%'"; \
  -e "CREATE USER 'repl'@'%' IDENTIFIED BY 'slavepass';" \
  -e "GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';" \
  -e "SHOW MASTER STATUS;"
```
Output:
```console
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+----------+--------------+------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+--------------------+----------+--------------+------------------+-------------------+
| mysql-bin-1.000003 |      595 |              |                  |                   |
+--------------------+----------+--------------+------------------+-------------------+
```

### 5.2 Slave

Selanjutnya mengkonfigurasi **slave nodes**


***[Optional]*** Jika ingin menggunakan semisynchronous replication, Jalankan perintah berikut pada terminal:

```
$ docker exec -it slave mysql -uroot -psalupa \
    -e "INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';" \
    -e "SET GLOBAL rpl_semi_sync_slave_enabled = 1;" \
    -e "SHOW VARIABLES LIKE 'rpl_semi_sync%';"
```
***[Optional]*** Output dari semisynchronous setup pada slave

```console
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | ON    |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | ON    |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
```

Ubah (Jika berbeda) koordinat replikasi yang dilakukan pada tahap sebelumnya.

- **MASTER_LOG_FILE='mysql-bin-1.000003'**

Sebelum menjalankan perintah berikut ini:

```
$ docker exec -it slave mysql -uroot -psalupa \
    -e "CHANGE MASTER TO MASTER_HOST='master', MASTER_USER='repl', \
    MASTER_PASSWORD='slavepass', MASTER_LOG_FILE='mysql-bin-1.000003';"
```

Periksa status replication pada **slave node**
```
$ docker exec -it slave mysql -uroot -psalupa -e "SHOW SLAVE STATUS\G"
```

Output:

```console
*************************** 1. row ***************************
               Slave_IO_State: Checking master version
                  Master_Host: master
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin-1.000003
          Read_Master_Log_Pos: 595
               Relay_Log_File: slave-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin-1.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
                             ...
```

## 6. Masukan sebuah Data

Sekarang saatnya kita mencoba data replication dari master ke slave. Kita akan membuat sebuah
database dengan nama "MyDatabase".

Jalankan perintah berikut untuk membuat database dan menampilkan database pada master node:

```
$ docker exec -it master mysql -uroot -psalupa -e "CREATE DATABASE MyDatabase; SHOW DATABASES;"
```

Output:
```console
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| MyDatabase         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Jalankan perintah berikut untuk mengecek database yang telah di replicated pada slave node:

```
$ docker exec -it slave mysql -uroot -psalupa \
  -e "SHOW VARIABLES WHERE Variable_name = 'hostname';" \
  -e "SHOW DATABASES;"
```

Output:
```console
mysql: [Warning] Using a password on the command line interface can be insecure.
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| hostname      | slave |
+---------------+--------+
+--------------------+
| Database           |
+--------------------+
| information_schema |
| MyDatabase         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Sampai disini, kita telah berhasil mengkonfigurasi database replication pada Docker MySQL Images.


## 7. Membersihkan Environment: stop containers, menghapus network dan image *[Optional]*

Jika anda ingin membersihkan konfigurasi pada docker yang telah dilakukan, 
anda dapat mengikuti panduan berikut ini:

**Stop containers yang sedang berjalan**
```
$ docker stop master slave
```

**Hapus data directory yang dibuat (Lokasinya sama dengan tempat container dijalankan)**
```
$ sudo rm -rf d0 d1 d2
```

**Hapus network yang telah dibuat**
```
$ docker network rm replicanet
```

**Hapus MySQLI Image**
```
$ docker rmi mysql/mysql-server:8.0
```



## Referensi

- https://github.com/wagnerjfr/mysql-master-slaves-replication-docker
- https://github.com/wagnerjfr/docker-machine-master-slave-mysql-replication
