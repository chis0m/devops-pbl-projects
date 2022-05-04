# LAMP STACK IMPLEMENTATION

#### Project 1

Requirement
- Provision an Ubuntu server with AWS EC2 instance or any other VM of choice
- SSH into server through local terminal or cloudshell


## Update and Installation

```bash
# update packages before installation
sudo apt update

sudo apt install apache2

# verify apache is runing
sudo systemctl status apache2

```


#### Install MySQL
```bash
sudo apt install mysql-server

# run security script and follow instructions
sudo mysql_secure_installation
```


#### Install PHP
This will install the ubuntu default php version
```bash
sudo apt install php libapache2-mod-php php-mysql

# check php version
php -v
```


## Create Virtual host for your website

#### create directory for your website

```bash
sudo mkdir /var/www/projectlamp

# change ownership to current user
sudo chown -R $USER:$USER /var/www/projectlamp
```


#### create apache config file for your website

```bash
sudo vi /etc/apache2/sites-available/projectlamp.conf
```

#### paste the vhost config and save
```apache

<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

#### enable the configuration
```bash
sudo a2ensite projectlamp

# disable the default config
sudo a2dissite 000-default

# reload configuratin
sudo systemctl reload apache2
```


#### create index.html file to test

```bash

sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```


**Visit your public ip on the browser**


## Enable php on the website

#### change order of index files in dir.conf

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

```apache
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

```bash
sudo systemctl reload apache2
```

#### create an index.php to test
```bash
vim /var/www/projectlamp/index.php
```

**paste the following**

```php
<?php
phpinfo();
```

**Visit your public ip on the browser and see a list of php config**