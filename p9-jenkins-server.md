# CONTINUOUS INTEGRATION WITH JENKINS SERVER- Using Ubuntu

### Project 9

Continuation from [project 8](/p8-apache-load-balancer.md)

## Setup
![](https://soms-public-assets.s3.amazonaws.com/images/p9-servers.png)

Allow port 8080 in your security group

#### Install Jenkins

```bash
sudo apt update
sudo apt install default-jdk-headless

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins  
```

#### Setup Jenkins
1. visit jenkins server, in my case `http://100.25.218.43:8080`
2. ssh into the server and run `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
3. Login with the password
4. Click on install sugggested plugins
5. Get started


#### Setup github to work with jenkins
![](https://soms-public-assets.s3.amazonaws.com/images/p9_github_webhook.png)

The IP is the jenkins server public IP
**Note**: Remember to add the `/` at the end  of the webhook url. If not you might get `302` response

#### Setup a freestyle project on jenkins

1. Goto **Source Code Management**, **Build Triggers** and **Post-Build Actions** and do the following
![](https://soms-public-assets.s3.amazonaws.com/images/p9_configure_jenkins_1.png)
![](https://soms-public-assets.s3.amazonaws.com/images/p9_configure_jenkins_2.png)
![](https://soms-public-assets.s3.amazonaws.com/images/p9_configure_jenkins_3.png)

2. Update a file e.g README.md and commit, jenkins should be trigger and an project archived
![](https://soms-public-assets.s3.amazonaws.com/images/p9_artifacts_1.png)

You can also check `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/` in the server

## Jenkins Copy File to NFS Server

#### Setup jenkins
1. Install **Publish Over SSH** plugin
2. Goto **Manage Plugins** > **Configure System** > **SSH plugin configuration**
3. Add **ssh private key** used to connect to NFS Server, **Hostname** which is NFS server private ip, **username** of server, and remote directory **/mnt/apps**. Like so

![](https://soms-public-assets.s3.amazonaws.com/images/p9-jenkins-server-copy-ssh-config.png)

4. Goto Jenkins job/project configuration and add another **Post-build Action** and choose **Send Build artifacts over SSH** and save
![](https://soms-public-assets.s3.amazonaws.com/images/p9-build-artifact-over-ssh.png)

5. Goto to your repo and update the README.md file. It would trigger jenkins build automatically
6. Goto into your NFS server and run `cat /mnt/apps/html/README.md`

Note: if you get Permission denied from jenkins then go to NFS server and run `sudo chmod -R 777 /mnt/apps`, for this practice, that should be enough