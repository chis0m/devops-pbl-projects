# APACHE LOAD BALANCER - Using Ubuntu

### Project 8

Continuation from [project 7](/p7-nfs-server-and-devops-tooling.md)

#### servers
![](https://soms-public-assets.s3.amazonaws.com/images/p8-servers.png)

### Setup LB ubuntu server
1. provision and ec2 ubuntu instance
2. Allow port 80 on the security group
3. Run the following installments

```bash
sudo apt update
sudo apt install apache2 -y                                         #Install apache2
sudo apt-get install libxml2-dev

sudo a2enmod rewrite                                                #Enable following modules:
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

sudo systemctl restart apache2                                      #Restart apache2 service
```

#### configure the loadbalance
```bash
sudo vi /etc/apache2/sites-available/000-default.conf  
```

Add this configuration into this section `<VirtualHost *:80>  </VirtualHost>`
Don't forget to replace the ips below with your webservers private ip

```conf
<Proxy "balancer://mycluster">
    BalancerMember http://172.31.24.245:80 loadfactor=5 timeout=1
    BalancerMember http://172.31.21.66:80 loadfactor=5 timeout=1
    ProxySet lbmethod=bytraffic
    # ProxySet lbmethod=byrequests
</Proxy>

ProxyPreserveHost On
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```

#### visit the load balancer public ip
Replace with your lb ip

`http://54.87.181.148/index.php`

![](https://soms-public-assets.s3.amazonaws.com/images/p8-apache-lb-server.png)


#### Test traffic route by LB
1. In project 7 we mounted `/var/log/httpd` to `/mnt/logs` of NFS server, now unmount it
2. To do this edit your `/etc/fstab` like so

```text
UUID=ff8f80fc-0119-49a4-9445-98d1f1b0dc05	/	xfs	defaults	0	0

172.31.19.98:/mnt/apps 			/var/www        nfs     defaults        0       0

#172.31.19.98:/mnt/logs 			/var/log/httpd  nfs     defaults        0       0    # commented out this line

```

3. reboot the server
4. Tail the access logs of the two webservers `sudo tail -f /var/log/httpd/access_log`
5. Refresh the load balancer url few times

![](https://soms-public-assets.s3.amazonaws.com/images/p8-access_logs.png)