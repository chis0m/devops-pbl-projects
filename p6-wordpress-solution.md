# WEB SOLUTION WITH WORDPRESS - Using Rhel8.6

### Project 6

#### Lauch instances

1. Launch two EC2 instance with RedHat Linux Distro(wordpress and database)
2. On the security group of wordpress web server, allow port 80 from anywhere
3. On the security group of database server, allow port 3306 from the private ip of the webserver.

![](https://soms-public-assets.s3.amazonaws.com/images/p6_two_servers.png)
![](https://soms-public-assets.s3.amazonaws.com/images/p6_wordpress_server_sg.png)
![](https://soms-public-assets.s3.amazonaws.com/images/p6_database_sg.png)

#### Attach New Drives

1. For each instance, create at least 2 volumes of atleast 10g and attach to each instance. volumes a to f.
![](https://soms-public-assets.s3.amazonaws.com/images/p6_volumes.png)

#### Create Partitions on newly attached disks

Do the following for all newly attached

```bash
sudo lsblk                                                              # see volume list
sudo gdisk /dev/xvdf                                                    # begin formatting
```

```bash

GPT fdisk (gdisk) version 1.0.3

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): n                                                 # create a new partition
Partition number (1-128, default 1):
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8e00                  # use LVM hexcode
Changed type of partition to 'Linux LVM'

Command (? for help): w                                                 # write partiton to disk

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y                                        # accept partition
OK; writing new GUID partition table (GPT) to /dev/xvdh.
The operation has completed successfully.
```

#### create logical volume

```bash
sudo yum install lvm2
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1                          # create physical volumes
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1               # create volume group
sudo lvcreate -n apps-lv -L 14G webdata-vg                              # create logical volume apps-lv
sudo lvcreate -n logs-lv -L 14G webdata-vg                              # create logical volume logs-lv
sudo lsblk                                                                   
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv                               # format logical volumes
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

#### create mount points

```bash
sudo mkdir -p /var/www/html                                             # mountpoint for apps-lv

sudo mkdir -p /home/recovery/logs                                       # !importan - create directory for /var/logs backup before mount
sudo rsync -av /var/log/. /home/recovery/logs/                          # backup

sudo mount /dev/webdata-vg/apps-lv /var/www/html/                       # mount apps-lv
sudo mount /dev/webdata-vg/logs-lv /var/log                             # mount logs-lv

sudo rsync -av /home/recovery/logs/. /var/log                           # recover backup
```

#### Add logical volumes mount to fstab, like so

```bash
UUID=ff8f80fc-0119-49a4-9445-98d1f1b0dc05       /       xfs     defaults        0       0

# mount for wordpress webser
/dev/webdata-vg/apps-lv              /var/www/html     ext4     defaults         0        0
/dev/webdata-vg/logs-lv              /var/log          ext4     defaults         0        0
```

```bash
sudo mount -a                                                         # check mount validity
sudo systemctl daemon-reload
```

```bash
[ec2-user@wordpress-server ~]$ lsblk
NAME                     MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda                     202:0    0  10G  0 disk
├─xvda1                  202:1    0   1M  0 part
└─xvda2                  202:2    0  10G  0 part /
xvdf                     202:80   0  10G  0 disk
└─xvdf1                  202:81   0  10G  0 part
  └─webdata--vg-apps--lv 253:0    0  14G  0 lvm  /var/www/html
xvdg                     202:96   0  10G  0 disk
└─xvdg1                  202:97   0  10G  0 part
  ├─webdata--vg-apps--lv 253:0    0  14G  0 lvm  /var/www/html
  └─webdata--vg-logs--lv 253:1    0  14G  0 lvm  /var/log
xvdh                     202:112  0  10G  0 disk
└─xvdh1                  202:113  0  10G  0 part
  └─webdata--vg-logs--lv 253:1    0  14G  0 lvm  /var/log
```

### Do the same for the database server

mount points - `/db` and `/logs`

Result below

```bash
UUID=ff8f80fc-0119-49a4-9445-98d1f1b0dc05       /       xfs     defaults        0       0
/dev/database-vg/db-lv                   /db            ext4     defaults       0       0
/dev/database-vg/logs-lv                 /var/log       ext4     defaults       0       0
```

```bash
[ec2-user@database-server ~]$ lsblk
NAME                      MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda                      202:0    0  10G  0 disk
├─xvda1                   202:1    0   1M  0 part
└─xvda2                   202:2    0  10G  0 part /
xvdf                      202:80   0  20G  0 disk
└─xvdf1                   202:81   0  20G  0 part
  └─database--vg-logs--lv 253:0    0  18G  0 lvm  /var/log
xvdg                      202:96   0  20G  0 disk
└─xvdg1                   202:97   0  20G  0 part
  └─database--vg-db--lv   253:1    0  18G  0 lvm  /db
xvdh                      202:112  0  10G  0 disk
└─xvdh1                   202:113  0  10G  0 part
```

### Installation on webserver

#### Installation of php and apache

```bash
sudo yum -y update

sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json         # install php

sudo systemctl enable httpd                                             # start apache
sudo systemctl start httpd

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1

sudo systemctl restart httpd                                            # restart apache
```

#### Installation of wordpress

```bash
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz

sudo chown -R $USER:$USER wordpress                                     # change ownership

cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/

sudo chown -R apache:apache /var/www/html/wordpress                     # change ownership to apache
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

#### Installation of mysql client

```bash
sudo yum install mysql
```

### Installation on DB Server

#### Installation of mysql

```bash
sudo yum update
sudo yum install mysql-server

sudo systemctl restart mysqld
sudo systemctl enable mysqld

sudo mysql                                                              # create user for remote accesss
```

```sql
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `wordpress`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'wordpress';
GRANT ALL ON wordpress.* TO 'wordpress'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

### Access DB from webserver

```bash
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```

Results:

```bash
[ec2-user@wordpress-server ~]$ mysql -u wordpress -p -h 172.31.27.140
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.26 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| wordpress          |
+--------------------+
2 rows in set (0.01 sec)

mysql>
```

### Accessing wordpress

```bash
cd /var/www/html/wordpress

sudo vi wp-config.php
```

#### update database credentials like so

```php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'wordpress' );

/** Database hostname */
define( 'DB_HOST', '172.31.27.140' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

**Go to the browser**

```
http://<Web-Server-Public-IP-Address>/wordpress/
```

![](https://soms-public-assets.s3.amazonaws.com/images/p6_wordpress_solution.png)