# Install-MySQL-Cluster
Tutorial Install MySQL CLuster Dengan 3 Server di CentOS/RHEL 8

![Mysql](https://tilmaners.files.wordpress.com/2017/05/mysql-logo.jpg?w=648)

# Persiapan 	:

Sedia 3 Server centos 7
- 192.168.1.21	mysql1 (nama hostname)
- 192.168.1.23 	mysql2 (nama hostname)
- 192.168.1.24 	mysql3 (nama hostname)
  
--------------------------------------------------

# House Skiping 	:
Lakukan langkah yang sama di ke tiga server

1. Update Centos/RHEL 8 supaya mendapat package baru

```
sudo dnf update -y
```
2. edit selinux

```
vim /etc/selinux/config
```
disabled selinux
  
3. Disable Firewall

```
systemctl stop firewalld
```

```
systemctl disable firewalld
```
4. Reboot OS

```
reboot
```

5. Config etc host dengan edit etc/hosts

```
vim /etc/hosts
```

6. Tambahkan ip sebagai berikut di semua server

```
192.168.1.21	mysql1
192.168.1.23 	mysql2
192.168.1.24 	mysql3
```
  
7. test koneksi

```
Ping 192.168.1.21 mysql1
```
```
Ping 192.168.1.23 mysql2
```
```
Ping 192.168.1.24 mysql3	
```

# Install MySQL 8.0 
Lakukan langkah yang sama di ke tiga server

1. Add repolist Mysql 8.0

```
wget https://repo.mysql.com/mysql80-community-release-el7-1.noarch.rpm
```

2. Install repo

```
sudo yum localinstall mysql80-community-release-el7-1.noarch.rpm -y
```

3. Cek Versi Mysql

```
yum repolist enabled | grep "mysql.*-community.*"
```

4. Install MySQL server & MySQL SHELL

```
yum install mysql-community-server mysql-shell -y
```
5. Selesai Installasi di ketiga server

# Config InnoDB CLuster 3 Mesin

1. Start Mysql Server

```
systemctl start mysqld
```

2. ganti password root

```
grep passw /var/log/mysqld.log
```

3. masuk mysql shell

```
mysqlsh --sql root@localhost
```

4. Masukan password sementara tadi lalu tekan enter.

5. ganti password root

```
set password='password_root_kamu';
```

6. membuat user untuk cluster InnoDB

```
create user 'cluster_user_kamu' identified by 'password_kamu';
```
7. Berikan permision user cluster kamu

```
grant all privileges on *.* to 'cluster_user_kamu'@'%' with grant option;
```
8. reset master

```
reset master;
```

# Lakukan Konfigurasi Hanya di Master saja

1. Cara masuk MySQL Shell di dalam Shell

1.1. masuk ke mysql shell JS (Javascript)

```
\js
```
1.2. Catatan untuk masuk ke Command Line MySQL adalah sebagai berikut :

```
\sql
```
1.3. Tambahkan Ip Allow List untuk anda Konfigurasi Cluster InnoDB di Mode SQL Shell:


```
SET GLOBAL group_replication_ip_allowlist="192.168.1.0/24";
```

2. Configurasi instance by user biasa jangan root (kandidat innodb clusternya)

2.1. mysql1
- isi password user kamu di command shell Javascript dengan ketik perintah \js, lalu tambahkan instance pertamamu :
```
dba.configureInstance('cluster_user_kamun@mysql1')
```
- password enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

2.2. mysql2
- ambahkan instance keduamu :
```
dba.configureInstance('cluster_user_kamu@mysql2')
```
- save password ketik "y" lalu enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

2.3. mysql3
- ambahkan instance ketigamu :
```
dba.configureInstance('cluster_user_kamu@mysql3') 
```
- save password ketik "y" lalu enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

3. lalu jika selesai , keluar dari Shell Javascript untuk configurasi user biasa di cluster InnoDB nya

```
/q
```
3. Konfigurasi Cluster InnoDB MySQL

- masuk ke mysql shell cluster_user_kamu Master

```
mysqlsh cluster_user_kamu@mysql1
```

- Membuat nama cluster innodb di user biasa

```
cluster=dba.createCluster('nama_cluster_kamu')	
```

- cek status cluster di DB Master

```
cluster.status()
```

- Menambahkan instance mysql mesin ke 2 ke 3 dan sterusnya

4. Menambah jumlah instance pada CLuster InnoDB MySQL

- Menambah instance ke 2

```
cluster.addInstance('cluster_user_kamu@mysql2')
```

- Menambah instance ke 3

```
cluster.addInstance('cluster_user_kamu@mysql3')
```

5. Terakhir cek status cluster InnoDB

```
cluster.status()
```

Catatan R/W instance bisa CRUD kalau R/O instance hanya Melihat tanpa bisa mengubah isi database dari instance master

selesai


-------------------------------------------------------------------------------------------------------------------------------

# Kumpulan Command MySQL Cluster

1. Untuk Commant R/W smua

```
cluster.switchToMultyPrimaryMode()
```

2. Untuk Melihat group InnoDB cluster bisa masuk seagai root

```
mysql -u root -p
```
Password = password_root_kamu

```
select * from performance_schema.replication_group_members;
```

3. Untuk Command Import Database adalah

```
mysql -u root -p nama_database_kamu < database_kamu.sql
```

4. Untuk Command masuk mysql shell root adalah

```
mysqlsh --sql root@localhost
```
password = Password_root_kamu

5. Untuk Command masuk mysql shell user biasa

```
mysqlsh cluster_user_kamu@mysql1
```
Password = Password_user_kamu

```
cluster.status()
```

6. Untuk Command masuk mysql user biasa

```
mysql -u cluster_user_kamu -p
```
Passw0rd = Password_user_kamu

7. Untuk Command masuk cluster InnoDB

```
cluster=dba.getCluster()
```

8. Untuk Command melihat versi mysql

- Lewat Command Shell MySQL

```
mysqlsh --sql root@localhost
```
atau dengan masuk cli langsung

```
mysqlsh -u root -p
```

lalu ketikan di command line MySQL
```
SHOW VARIABLES LIKE "%version%";
```

- Tanpa leway Command Shell MySQL

```
mysql -v
```
9. Untuk Command mengganti primary instance mysql

- Login terlebih dahulu lewat Command Shell MySQL JS
```
mysqlsh clusteradmin@mysql1
```
- lalu ketikan di command line MySQL JS untuk mengganti primary instance tersebut
```
cluster.setPrimaryInstance('mysql1:3306')
```
