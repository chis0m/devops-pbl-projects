# DEVOPS TOOLING - Using Rhel8.6

### Project 7


## Prepare NFS Server
#### Spin up a new EC2 instance with RHEL Linux 8 Operating System.
#### Configure LVM on the Server
Note: Refer to [Project 6](https://github.com/chis0m/devops-pbl-projects/blob/master/p6-wordpress-solution.md)
You should have the following results

```sql
sudo mysql

mysql> CREATE DATABASE tooling;
mysql> CREATE USER `webaccess`@`172.31.16.0/20` IDENTIFIED BY 'password';         # Allows connection from only this subnet 172.31.16.0/20
mysql> GRANT ALL ON tooling.* TO 'webaccess'@`172.31.16.0/20`;                   # Allows connection from only this subnet 172.31.16.0/20
mysql> FLUSH PRIVILEGES;
mysql> SHOW DATABASES;
mysql> exit
```

##### storage location
```bash
tree /dev/store-vg/
```
```text
/dev/store-vg/
├── lv-apps -> ../dm-0
├── lv-logs -> ../dm-2
└── lv-opts -> ../dm-1
```

##### List disk
```bash
sudo lsblk -f
```
```text
NAME                   FSTYPE      LABEL UUID                                   MOUNTPOINT
xvda
├─xvda1
└─xvda2                xfs         root  ff8f80fc-0119-49a4-9445-98d1f1b0dc05   /
xvdf
└─xvdf1                LVM2_member       Y7xJjs-vzNH-CMXo-PxDY-rkZX-1K66-QcFIov
  └─store--vg-lv--opts xfs               0bc2dac4-20ae-4c5d-9a35-ffc27fa7b8c1   /mnt/logs
xvdg
└─xvdg1                LVM2_member       IDlCvC-MfGO-FRYb-OzyF-Jmc2-ISCZ-99TcVZ
  ├─store--vg-lv--apps xfs               7d4bf3b4-114a-43c8-b37b-55cb7d415fc5   /mnt/apps
  ├─store--vg-lv--opts xfs               0bc2dac4-20ae-4c5d-9a35-ffc27fa7b8c1   /mnt/logs
  └─store--vg-lv--logs xfs               4e640672-7e65-41c9-a631-4d322d372e4c   /mnt/logs
xvdh
└─xvdh1                LVM2_member       iLMCkC-gdzX-cc5P-bDGo-p7If-1QjA-0Ov9Hc
  └─store--vg-lv--apps xfs               7d4bf3b4-114a-43c8-b37b-55cb7d415fc5   /mnt/apps
```

##### volume mount
```bash
cat /etc/fstab
```
```text
UUID=ff8f80fc-0119-49a4-9445-98d1f1b0dc05	/	xfs	defaults	0	0
/dev/store-vg/lv-apps                        /mnt/apps  xfs     defaults        0       0
/dev/store-vg/lv-logs                        /mnt/logs  xfs     defaults        0       0
/dev/store-vg/lv-opts                        /mnt/logs  xfs     defaults        0       0
```


#### Install NFS server and configure
```bash
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service


sudo chown -R nobody: /mnt/apps                                                                 # set permissions
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

#### Edit nfs exports file

```bash
sudo vi /etc/exports 
```

paste the following
```text
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
```
**Note:** *172.31.16.0/20* is subnet cidr

```bash
sudo exportfs -arv                                                                              # activate nfs

rpcinfo -p | grep nfs
```

output
```text
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
```
#### configure security groups
`TCP 111` `UDP 111`  `UDP 2049`

![](https://soms-public-assets.s3.amazonaws.com/images/p7-security-group-config.png)


### Configure database server
Note: Refer to [Project 6](https://github.com/chis0m/devops-pbl-projects/blob/master/p6-wordpress-solution.md)


### Prepare the Web Servers

#### NFS Client
Note: Replace  `172.31.19.98` with your NFS server private IP
Launch new EC2 instance
```bash
sudo yum install nfs-utils nfs4-acl-tools -y                                                   # install NFS client

sudo mkdir /var/www                                                                            # Mount /var/www/ and target the NFS server’s export for apps
sudo mount -t nfs -o rw,nosuid 172.31.19.98:/mnt/apps /var/www                                 #  Mount to NFS server export for apps 
sudo mount -t nfs -o rw,nosuid 172.31.19.98:/mnt/logs /var/log/httpd                           # Mount to NFS server export for logs 

df -h                                                                                          # verify NFS was mounted successfully

sudo vi /etc/fstab                                                                             # persist the mounting
```

paste the following
```text                                              
172.31.19.98:/mnt/logs     /var/log/httpd  nfs     defaults        0       0
172.31.19.98:/mnt/apps     /var/www        nfs     defaults        0       0
```

#### Create a second server for NFS client
Do same configuration as above
![](https://soms-public-assets.s3.amazonaws.com/images/p7-servers.png)

Console showing successful NFS mount
![](https://soms-public-assets.s3.amazonaws.com/images/p7-nfs-clients-console.png)

#### web server installation

```bash
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1
```

#### Disable SeLinux
```bash
sudo vi /etc/sysconfig/selinux                                                                          # edit and disable SeLinux                                                                                                                            
```
change `SELINUX=enforce` to `SELINUX=disabled`

#### setup website

```bash
cd ~
git clone https://github.com/darey-io/tooling.git

sudo cp -r tooling/html/* /var/www/html

cd /var/www/html

sudo vi function.php                                                                                    # edit and update function.php
```

```php
// connect to database
$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');                   # Old code

// connect to database                      
$dbHost = '172.31.25.243';                                                                              # Replace with this
$dbName = 'tooling';
$dbUserName = 'webaccess';
$dbPassword = 'password';
$db = mysqli_connect($dbHost, $dbUserName, $dbPassword, $dbName);
```


#### Create new user
user with username: `myuser` and password: `password`
**Note:** the password is encrypted with md5 below

```sql
mysql> INSERT INTO `users` (`id`, `username`, `password`, `email`, `user_type`, `status`) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
```


#### Visit website from two servers created

![](https://soms-public-assets.s3.amazonaws.com/images/p7-web-server-1.png)

![](https://soms-public-assets.s3.amazonaws.com/images/p7-web-server-2.png)