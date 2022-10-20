# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

AIM: Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accommodate increased traffic and at the same time, has reasonable cost.

## Summary
To achieve our above aim we will need
- Domain Name Service(Route53)
- Virtual Network with access to internet (Virtual Private Cloud with Internet Gateway - VPC and IG)
- Bastion Hosts (Jump Box)
- Proxy Servers (using Nginx)
- NAT Gateway (NGW)
- Load Balancers(Application Load Balancer - ALB)
- Automatic scalers (Auto Scaling Groups - ASG)
- EFS server (if WebServer is Worpress)
- Databse with Read Replica (Aurora) - both Tooling and Wordpress access this
  
We will setup a VPC with 2 Availability Zones, 8 Subnets(2 Public and 6 Private), 2 ALBs (1 public and 1 private), 4 ASG (1 for proxy server, 1 for Bastion and 2 for the webservers i.e 1 for Tooling and 1 for Wordpress app).

#### HTTP Traffic FLow
`Internet -> Route53 -> Internet Gateway -> Public ALB (with Listener on port 80/443)-> ASG Target Groups (Housing Proxy Servers) -> The Proxy Servers -> Private ALB -> ASG(Housing Webservers) -> The WebServers -> Database or EFS`

### SSH Traffic
`localhost -> Bastion -> Proxy Servers and Webservers`

### Network Address Translation (NAT) Traffic flow
`Webservers and ProxyServers -> NGW(in public subnet) -> Internet`


Tags are Important
- Case: CamelCase plus Kebab-Case
- Pattern: <Project>-<OwningResource(optional)>-<Resource> e.g MC-Wordpress-SG, MC-ExtALB-SG
- Environment: Dev, Test, Prod

### Steps

#### VPC
- IP: 10.0.0.0/16

#### 8 Subnets
In the 2 Public Subnets (1 for each AZ) we will have
- Name: `MC-PublicSubnets` (1 & 2)
- IP: 10.0.1.0/24 and 10.0.2.0/24
- The bastion hosts
- 1 ASG
- 1 public ALB
- 1 NGW

In 2 Private subnet (1 for each AZ), we will have
- Name: `MC-ProxyPrivateSubnet`(1 & 2)
- IP: 10.0.3.0/24  and 10.0.4.0/24 
- The 2 proxys servers
- 1 ASG

In 2 Private subnet (1 for each AZ), we will have
- Name: `MC-WebServerPrivateSubnet` (1 & 2)
- IP: 10.0.5.0/24 and 10.0.6.0/24 
- The Webservers (Tooling and Wordpress)
- 2 ASG (1 for each type of the webservers)
- 1 ALB

In 2 Private subnet (1 for each AZ), we will have
- Name: `MC-DatabasePrivateSubnet` (1 & 2)
- IP: 10.0.7.0/24 and 10.0.8.0/24 
- The dabases - Write in one AZ and Read Replica in another AZ

#### Gateways
- Name: `MC-IGW`
- Create Internet and Nat gate ways (IGW and NAT)
- Attach IGW to the Public Route Table
- Attach NAT GW to the route table to be connected to Proxy and Webserver Subnet
Internet Gateway
- Will be attached to (specified in) the RouteTable which is associated with the 2 PublicSubnets
Nate Gateway
- Name: `MC-NATGW`
- The NGW will be hosted in any of the public subnets but will be attached to (specified in) the RouteTable which is associated with the 2 ProxyPrivateSubnet and 2 WebServerPrivateSubnet. In other words, the above 4 subnets will be associated to same routetable. This is because devices in this subnets needs to have `one-way` traffic to the internet for updates and downloading packages.
- Allocate Elastic IP
- Destination: 0.0.0.0, Target: nat-created

#### Route Tables (RT)
Public RT
- IGW: Desitination - 0.0.0.0, Target - IGW
- local

Proxy and Webserver RT
- NATGW: Destination - 0.0.0.0, Target - NAT
- local

Database RT
- local: Destination - 10.0.0.0/16, Target - local

#### Security Groups
Bastion SG
- Name: `MC-Bastion-SG`
- Allow IPs of all DevOps Engineers or if you have a VPN, whitelist its IP
- For instance run `curl www.canhazip.com` to get your public IP and then whitelist it
- Note: Ensure your IP is static unless you are using a VPN IP

Public ALB SG
- Name: `MC-PublicALB-SG`
- HTTP/HTTPS: Allow from 0.0.0.0 (internet)

Proxy server SG
- Name: `MC-ProxyServer-SG`
- Allow SSH traffic from Bastion SG
- Allow HTTP and HTTPS from ALB SG

Private ALB SG
- Name: `MC-PrivateALB-SG`
- HTTP and HTTPS: Allow access from Proxy Server SG

Webserver SG
- Name: `MC-Webserver-SG`
- Allow SSH traffic from Bastion SG
- Allow HTTP and HTTPS from Private ALB SG

#### Bastion Launch Template, ASG, Target Group#
Bastion Launch Template
- Select ubuntu AMI
- Select key pair
- Subnet: You don't need to specify but if you want to then - Select `MC-PublicSubnet 1` Subnet. The other will be set when configuring ASG using this Template
- Security Group - Select `MC-Bastion-SG`
- Add Tags
- Leave everything else as default
  
Bastion ASG
- Select Bastion Launch template
- Select the two public subnets
- Select `No Load Balancer`. But if we are to need a load balancer, it would be a Network Load balancer since Bastion for to SSH access into Private resource
- Leave everything else as default
- After create, if the Bastion doesnt have a public IP, then it means you forgot to check the `Enable Auto-assign public Ip` in your public subnet settings

#### Target Groups

Target Groups for Proxy Server
- Name: `MC-ProxyServer-TG`
- TG type should be `instances`
- Select correct VPC
- Add tags
- Leave everyother thing as default

Webserver Tooling Target Group
- Name: `MC-ToolingAppServer-TG`
- Target Type: Select Instances
- Select correct VPC
- Add tags
- Leave everything as default

Webserver WordPress Target Group
- Name: `MC-WordpressAppServer-TG`
- Target Type: Select Instances
- Select correct VPC
- Add tags
- Leave everything as default

#### Load Balancers

Public/Internet Facing ALB
- Name: `MC-PublicLB-ALB`
- Configure it to listen on ports 80 and 443
- Select `internet facing`
- Select correct VPC
- Subnet: Slect the 2 AZs and` MC-PublicSubnet 1& 2` respectively
- Security Group: Select the `MC-PublicALB SG`
- Point port 80/443 to `MC-ProxyServer-TG`. But 443 will need SSL cert configuration so for now you can ignore it
- Leave everything as default
- Create LB
- 
Webserver Private ALB
- Name: `MC-WebserverInternal-ALB`
- Select `internal`
- Select correct VPC
- Subnet: Slect the 2 AZs and`MC-WebserverPrivateSubnet 1 & 2` respectively
- Security Group: Select the `MC-Webserver-SG`
- Point port 80 to `MC-WordpressAppServer-TG`. 443 will need SSL cert configuration so for now you can ignore it
- Leave everything as default
- Create LB
- After creation go and edit the listener and add the following rules.
- Forward Traffic: HTTP 80 -> Webserver Target Group
  - Create a rule: `Host Header` - tooling.app.<domain>.com -> THEN Tooling App Target Group
  - Create a rule: `Host Header` - wordpress.app.<domain>.com -> THEN Wordpress App Target Group
  - Default rule: `If Requests otherwise not routed` -> THEN 503: Oops, I think you are lost


Create an AMI Image of the proxy ser
- Create the EC2 instance with RedHat OS
- SSH into the server and 
- Install Nginx
  
```bash
#!/usr/bin/bash
sudo yum update -y
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-7.rpm
sudo yum install -y nginx git
sudo systemctl restart nginx
ssudo systemctl enable nginx

# Configure Nginx
sudo nano /etc/nginx/conf.d/nginx.conf
```
- Add the following rule

```nginx

# Also comment out the default nginx server block

...

    server {
        listen 80;
        server_name www.tooling.app.chisomejim.click;
        location / {
            proxy_pass internal-MC-WebserversPrivate-ALB-1362305892.us-east-1.elb.amazonaws.com;
            proxy_set_header Host $host;
        }
    }

    server {
        listen 80;
        server_name www.wordpress.app.chisomejim.click;
        location / {
            proxy_pass internal-MC-WebserversPrivate-ALB-1362305892.us-east-1.elb.amazonaws.com;
            proxy_set_header Host $host;
        }
    }

...    

```
- Creat an AMI Image


#### Proxy Server Launch Template and ASG

Proxy Server Launch Template
- Name: `MC-ProxyServer-LT`
- We will use the Launch Template when creating the Proxy server ASG
- Use the Proxy Server Custom AMI created above or you can use a fresh Redhat AMI and t2.medium
- Choose your keypair
- Take Note: In an ideal situation this ssh keypair is only used by the master engineer. Everyother engineer would have their own public keep populated across the server. Here `ansible, puppet, chef config managers` can help with this
- Select subnet as `MC-ProxyServerSubnet 1`. When creating ASG we will also specify the second subnet, by that we would scale across the two AZs
- Select Security group as the `MC-ProxyServer-SG` earlier created
- Add Tags
- Leave everything else as default

Note: When you make changes to your Lauch template and a new version is created, always remember to pick `Later version` not default in the version section

Proxy Server ASG
- Choose the `MC-ProxyServer-LT` and Version should be `Latest(n)` where n is the latest number
- Select the VPC
- Select subnets `MC-ProxyServerSubnet 1 & 2`
- Attach existing Load Balancer. Choose the Public/Internet Facing Load Balancer
- **Note:** In attaching a Load Balancer to an ASG, it is actually the Target Group the Load balancer is forwarding-to that you are choosing
- Group Seize: Minimum - 1, Desired Capacity/Maximum - 2
- Scaling Policies: Metric Type -> CPU, Target Value -> 70

#### AMI for Tooling and Webserver

AMI for Tooling and Wordpress Webservers
- Create RedHat EC2 instance
- Install Nginx with same instructions for Proxy Server
- For demo purposes, Create a HTML content you like and paste in `/usr/share/nginx/html/index.html`. However you can go further and setup an actually app 
- Create AMI for each `MC-ToolingApp-AMI` and `MC-WordpressApp-AMI`


#### Lauch Templates and ASG for the Webservers
Tooling and Wordpress Web Server Launch Template (LT)
- Name: `MC-ToolingApp-LT` and `MC-WordpressApp-LT`
- Do same for each accordingly. Same subnet though
- Select Wordpress(Tooling, if tooling LT) AMI
- Select key pair
- Select `MC-WebserverPrivateSubnet 1` Subnet. The other will be set when configuring ASG using this Template
- Security Group - Select `MC-Webserver-SG`
- Add Tags
- Leave everything else as default

Tooling and Wordpress ASG
- Name: `MC-ToolingApp-ASG` and `MC-WordpressApp-ASG`
- Do same for both accordingly
- Choose the `MC-WordpressApp-LT` or `MC-ToolingApp-LT` accordingly
- Select the VPC
- Select same subnets `MC-WebserverPrivateSubnet 1 & 2`
- Attach existing Load Balancer. Choose the `MC-WebserverInternal-ALB`, select `MC-ToolingApp-TG` or `MC-WordpressApp-TG` accordingly
- Group Seize: Desired Capacity/Minimum - 1, Maximum - 3
- Scaling Policies: Metric Type -> CPU, Target Value -> 70


#### Domain Setup
Route53 
- will resolve the Domain name.
- Create a CNAME to point to the public ALB DNS address
- You can visit `dnschecker.org` to very your the cname works appropriately
- If you bought domains from third party (lets use namecheap), then
  - Click `create hosted zones` and fill up the following
  - Domain Name:  domain you paid for e.g `chisomejim.click`
  - Type: choose Public hosted zone and create
  - Two records will be created. NS(Name servers) and SOA to be mapped to the Domain Name in namecheap
  - Copy Name Server values and paste in namecheaps Domain `Manage -> Nameserver -> Custom DNS`


### Possible Errors
- On using RedHat Based Distros of Proxy Server, you might get this error `(13: Permission denied) while connecting to upstream`
- Solution: execute `sudo setsebool -P httpd_can_network_connect 1` on the proxy server. At this point you might want to rebuild a new AMI image with this comand executed already


#### Part 2
- Setup Database Servers in `MC-Default-Subnet`
- EFS Setup in `MC-WebserverPrivate-Subnet 1 & 2` must be in same subnet as the application connecting to it
- Actually wordpress App installation and connection to EFS
- TLS Certificate

To be continued