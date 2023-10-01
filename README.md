# Домашнее задание к занятию "Репликация и масштабирование. Часть 1" - Наталья Мартынова (Пономарева)

### Задание 1.
Основное отличие Master-Master от Master-Slave в возможности выполнять запись в обе базы одновременно, что позволяет увеличить доступность и эффективность работы кластера БД, однако данная схема увеличивает вероятность возникновения split brain и не увеличивает отказоустойчивость`

### Задание 2.

Master IP: 192.168.0.105
Slave IP:  192.168.0.106

1. Install MySQL
```
sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo yum install mysql-community-server
sudo systemctl enable mysqld
sudo systemctl start mysqld
sudo grep 'temporary password' /var/log/mysqld.log
mysql_secure_installation
```
2. Setting Master
```
sudo nano /etc/my.cnf
bind-address           = 192.168.0.105
server-id              = 1
log_bin                = mysql-bin
log_bin = /var/log/mysql/mysql-bin.log 
log_bin_index =/var/log/mysql/mysql-bin.log.index 
relay_log = /var/log/mysql/mysql-relay-bin 
relay_log_index = /var/log/mysql/mysql-relay-bin.index
sudo systemctl restart mysqld
mysql -uroot -p
CREATE USER 'user'@'192.168.0.107' IDENTIFIED BY 'Qwerty123+';
GRANT REPLICATION SLAVE ON *.* TO 'user'@'192.168.0.107' IDENTIFIED BY 'Qwerty123+';
FLUSH PRIVILEGES;
FLUSH TABLES WITH READ LOCK;
mysqldump -u root -p --all-databases --master-data > dbdump.sql
SHOW MASTER STATUS\G
scp dbdump.sql 192.168.0.107:/tmp
```
![Снимок1](https://github.com/NatoshFehn/hw-rbd-06/blob/main/Снимок1.png)

3. Setting Slave
```
sudo nano /etc/my.cnf
bind-address           = 192.168.0.107
server-id              = 2
log_bin                = mysql-bin
log_bin = /var/log/mysql/mysql-bin.log 
log_bin_index =/var/log/mysql/mysql-bin.log.index 
relay_log = /var/log/mysql/mysql-relay-bin 
relay_log_index = /var/log/mysql/mysql-relay-bin.index
sudo systemctl restart mysqld
mysql -u root -p < /tmp/dbdump2.sql
STOP SLAVE;
CHANGE MASTER TO
MASTER_HOST='192.168.0.106',
MASTER_USER='replicauser',
MASTER_PASSWORD='ehSexpYt95N',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=1104;
START SLAVE;
SHOW SLAVE STATUS;
```
4. Test replication
```
#Master
UNLOCK TABLES;
SHOW DATABASES;
SHOW PROCESSLIST;
CREATE DATABASE replicatest;
#Slave
SHOW PROCESSLIST;
SHOW DATABASES;
```
![Снимок2](https://github.com/NatoshFehn/hw-rbd-06/blob/main/Снимок2.png)

![Снимок3](https://github.com/NatoshFehn/hw-rbd-06/blob/main/Снимок3.png)

![Снимок4](https://github.com/NatoshFehn/hw-rbd-06/blob/main/Снимок4.png)

Reset replication
```
#slave
STOP SLAVE;
RESET SLAVE;
```
