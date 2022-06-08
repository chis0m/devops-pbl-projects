# LOAD BALANCER SOLUTION WITH NGINX- Using Ubuntu

### Project 10
![](https://soms-public-assets.s3.amazonaws.com/images/p10_nginx_lb.png)

![](https://soms-public-assets.s3.amazonaws.com/images/p10_servers.png)

Allow port 80 and 443

#### Configure Nginx as Load Balancer
1. Instatiate an Ubuntu EC2 Instance
2. Edit `/etc/hosts` like so
```hosts
127.0.0.1 localhost
54.163.198.11 web1
54.227.72.108 web2
```
#### Install Nginx
```bash
sudo apt update
sudo apt install nginx
```

#### Add load balancing script

```bash
sudo vi /etc/nginx/nginx.conf
```

paste the following

```nginx
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
}

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
}
```

Comment out `include /etc/nginx/sites-enabled/*;`

```bash
sudo systemctl restart nginx
sudo systemctl status nginx
```

#### New Domain
1. Goto a domain provider and register a new domain e.g mine is `chisomejim.link`
2. Create an elastic ip and attach to the load balancer
![](https://soms-public-assets.s3.amazonaws.com/images/p-10-elastic-ip.png)
3. Check that your Web Servers can be reached from your browser
![](https://soms-public-assets.s3.amazonaws.com/images/p10-chisomejim.link.png) 

#### Install certbot

```bash
sudo systemctl status snapd                                 # make sure snapd is running 
sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot               # create symlink      

sudo certbot --nginx                                        # follow the prompt to create ssl certs
```
Typically nginx will crawl through the configuration files - nginx.conf - to get the domanin to secure

Visit your site with https. You should see the padlock at the top
![](https://soms-public-assets.s3.amazonaws.com/images/p-10-ssl-cert-https.png)


#### Renew Certificate
By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently

```bash
crontab -e                                                 # edit the user crontab file
```

paste this

```
* * * */1 *  root /usr/bin/certbot renew > /dev/null 2>&1 
```
Schedule to renew every month