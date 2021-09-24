# Install-MySQL-Cluster
Tutorial Install MySQL CLuster Dengan 3 Server di CentOS 7

# Persiapan 	:

Sedia 3 Server centos 7
- 192.168.1.21	mysql1 (nama hostname)
- 192.168.1.23 	mysql2 (nama hostname)
- 192.168.1.24 	mysql3 (nama hostname)
  
--------------------------------------------------

# House Skiping 	:
Lakukan langkah yang sama di ke tiga server

1. Update Centos 7 supaya mendapat package baru

```
yum update -y
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
yum localinstall mysql80-community-release-el7-1.noarch.rpm -y
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
set password='Complicate3$';
```

6. membuat user clusteradmin

```
create user 'clusteradmin' identified by 'Fred123$';
```
7. permision user clusteradmin

```
grant all privileges on *.* to 'clusteradmin'@'%' with grant option;
```
8. reset master

```
reset master;
```

# Lakukan Konfigurasi Hanya di Master saja

# 1. masuk ke mysql shell JS (Javascript)

```
\js
```
Catatan untuk masuk ke Command Line MySQL adalah sebagai berikut :

```
\sql
```

2. Configurasi instance by user biasa jangan root (kandidat innodb clusternya)

2.1. mysql1
- isi password user biasa
```
dba.configureInstance('clusteradmin@mysql1')
```
- password enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

2.2. mysql2
- isi password user biasa
```
dba.configureInstance('clusteradmin@mysql2')
```
- save password ketik "y" lalu enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

2.3. mysql3
- isi password user biasa jangan root
```
dba.configureInstance('clusteradmin@mysql3') 
```
- save password ketik "y" lalu enter
- Do you want to perform the required configuration changes? [y/n] : y
- Do you want to restart the instance configuration it? [y/n] : y

3. lalu keluar untuk configurasi user biasa custer InnoDB nya

```
/q
```
3. Konfigurasi Cluster InnoDB MySQL

- masuk ke mysql shell clusteradmin Master

```
mysqlsh clusteradmin@mysql1
```

- Membuat nama cluster innodb di user biasa (clusteradmin)

```
cluster=dba.createCluster('lefredCluster')	
```
Catatan : lefred = nama clusternya

- cek status cluster di DB Master

```
cluster.status()
```
- Menambahkan instance mysql mesin ke 2 ke 3 dan sterusnya

4. Menambah jumlah instance pada CLuster InnoDB MySQL

- Menambah instance ke 2

```
cluster.addInstance('clusteradmin@mysql2')
```

- Menambah instance ke 3

```
cluster.addInstance('clusteradmin@mysql3')
```

5. Terakhir cek status cluster InnoDB

```
cluster.status()
```

Catatan R/W instance bisa CRUD kalau R/O instance hanya Melihat tanpa bisa mengubah database

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
Password = Complicate3$

```
select * from performance_schema.replication_group_members;
```

3. Untuk Command Import Database adalah

```
mysql -u root -p unhan_library < unhan_library.sql
```

4. Untuk Command masuk mysql shell root adalah

```
mysqlsh --sql root@localhost
```
password = Complicate3$

5. Untuk Command masuk mysql shell user biasa

```
mysqlsh clusteradmin@mysql1
```
Password = Fred123$

```
cluster.status()
```

6. Untuk Command masuk mysql user biasa

```
mysql -u clusteradmin -p
```
Passw0rd = Fred123$

7. Untuk Command masuk cluster InnoDB

```
cluster=dba.getCluster()
```

