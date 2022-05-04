# LEMP STACK IMPLEMENTATION (Linux, Nginx, MySQL, PHP)

#### Project 2

##### Requirements
- Provision an Ubuntu server with AWS EC2 instance or any other VM of choice
- SSH into server through local terminal or cloudshell

## Update and Installation

```bash
# update packages before installation
sudo apt update

sudo apt install nginx

# verify nginx is runing
sudo systemctl status nginx
```

#### Visit public ip
```bash
# Get your public ip
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

```bash
http://<Public-IP-Address>
```
**Should load Nginx welcome page**


#### Install MySQL
```bash
sudo apt install mysql-server

# run security script and follow instructions
sudo mysql_secure_installation
```


#### Install PHP-FPM
This will install the ubuntu default php version
```bash
sudo apt install php-fpm php-mysql

# check php version
php -v
```

## Configuring Nginx to use php-fpm

#### create root directory for your website

```bash
sudo mkdir /var/www/projectLEMP

# change ownership to current user
sudo chown -R $USER:$USER /var/www/projectLEMP
```

#### configure nginx to load your site

```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```

```nginx

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

```

#### enable your sites nginx config
```bash
# first check if config is correct
sudo nginx -t

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

# disable default nginx config
sudo unlink /etc/nginx/sites-enabled/default

# reload nginx
sudo systemctl reload nginx
```

#### create index.html file to test

```bash

sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

**Visit your public ip to test**

## Testing php with nginx

#### create php test file

```bash
sudo nano /var/www/projectLEMP/info.php
```

**paste**

```php
<?php
phpinfo();
```

**Visit your public ip to test**


## Retrieving data from mysql

#### login
```bash
sudo mysql
```
#### run sql queries
```sql
mysql> CREATE DATABASE `example_database`;

-- creat user with password
mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

-- Grant the user permission to the database created
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';

mysql> exit
```

#### login to the newly created database with the credentials

```bash
mysql -u example_user -p
```

```sql

mysql> SHOW DATABASES;

-- set the database to use
mysql> use example_database;

-- create table
mysql> CREATE TABLE todo_list (item_id INT AUTO_INCREMENT, content VARCHAR(255), PRIMARY KEY (item_id));

-- insert into the table
mysql> INSERT INTO todo_list (content) VALUES ("My first important item");

-- select to see the items in the table
mysql>  SELECT * FROM todo_list;

mysql> exit

```

#### create php script that would connect to your database

```bash
nano /var/www/projectLEMP/todo_list.php
```

**paste the following**

```php

<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

```

**Visit your public domain to see**

```http
http://<Public_domain_or_IP>/todo_list.php
```