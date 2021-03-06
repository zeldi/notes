# INSTALL VSFTPD + MYSQL + PAM-MYSQL EN AWS EC2 

OS: Amazon Linux AMI release 2016.09
Based: https://www.howtoforge.com/virtual-hosting-with-vsftpd-and-mysql-on-ubuntu-12.04
Config Passive mode: http://stackoverflow.com/questions/7052875/setting-up-ftp-on-amazon-cloud-server

## Steps

* Install pre-requisites packages:

```
yum install make gcc-c++ autoconf automake libtool rpm-build
yum pam-devel
yum install cyrus-sasl-devel
yum install mysql mysql-server mysql-devel
```

* Build module PAM

```
wget ftp://mirror.switch.ch/pool/4/mirror/fedora/linux/releases/23/Everything/source/SRPMS/p/pam_mysql-0.7-0.20.rc1.fc23.src.rpm
rpmbuild --rebuild /home/ec2-user/pam_mysql-0.7-0.20.rc1.fc23.src.rpm   
```

```
yum install ./rpmbuild/RPMS/x86_64/pam_mysql-0.7-0.20.rc1.amzn1.x86_64.rpm
```

```
service mysqld start
/usr/libexec/mysql55/mysql_secure_installation
```

* Configure Mysql

```
[root@TerraAPP-APP1 ~]# mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.5.52 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE vsftpd;
Query OK, 1 row affected (0.00 sec)

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON vsftpd.* TO 'vsftpd'@'localhost' IDENTIFIED BY 'ftpdpass';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON vsftpd.* TO 'vsftpd'@'localhost.localdomain' IDENTIFIED BY 'ftpdpass';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> USE vsftpd;
Database changed
mysql> CREATE TABLE `accounts` (
    -> `id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY ,
    -> `username` VARCHAR( 30 ) NOT NULL ,
    -> `pass` VARCHAR( 50 ) NOT NULL ,
    -> UNIQUE (
    -> `username`
    -> )
    -> ) ENGINE = MYISAM ;
Query OK, 0 rows affected (0.01 sec)

mysql> quit;
Bye
```

* Final step

```
useradd --home /home/vsftpd --gid nobody -m --shell /bin/false vsftpd 
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_orig
```

* Config File

```
vim /etc/vsftpd/vsftpd.conf
``` 

```
listen=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
nopriv_user=vsftpd
chroot_local_user=YES
secure_chroot_dir=/var/ftp/pub
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
guest_enable=YES
guest_username=vsftpd
local_root=/data/sites/www.$USER
user_sub_token=$USER
virtual_use_local_privs=YES
user_config_dir=/etc/vsftpd/vsftpd_user_conf
pasv_enable=YES
pasv_address="Your Public IP"
pasv_min_port=1024
pasv_max_port=1048

```

* Restart service

```
service vsftpd restart
```
