# ANSIBLE CONFIGURATION MANAGEMENT

### Project 11

![](https://soms-public-assets.s3.amazonaws.com/images/p11_jenkins_ansible.png)

Continuation from Project [9](/p9-jenkins-server.md) and [10](/p10-nginx-load-balancer.md)

#### Setup Repo
In your GitHub account create a new repository and name it `Ansible-config-mgt`

#### Setup Ansible
In same server as as Jenkins run

```bash
sudo apt update

sudo apt install ansible
```

#### Setup Jenkins
- Just like in [Project 9](/p9-jenkins-server.md), configure a Freestyle Project to build your `ansible` artifact.
- Configure Webhook in GitHub and set webhook to trigger ansible build.
- Configure a Post-build job to save all (**) files, like you did it in Project 9
- Update README.md on your master branch and see if the build starts automatically
- Run `ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` to confirm

**Note:** Trigger Jenkins project execution only for `master` branch.


### Begin Ansible Development
On the github repository `Ansible-config-mgt`
 
1. Create a new branch `feature-project-11 `
2. create playbook `playbooks/common.yml`
3. create inventories `inventory/dev.yml`,  `inventory/dev.yml`,  `inventory/staging.yml` and  `inventory/prod.yml`
4. update inventory/dev.yml

```yml
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```


#### Setup SSH agent for ansible
In your localhost terminal run
```bash
eval $(ssh-agent)
ssh-add ~/.ssh/<private-key>
ssh-add -l                                                                        # lists the added keys

ssh -A ubuntu@<public-ip-address>
```


#### Update the playbook/common.yml
paste the following
```yml
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

#### push update to github

```bash
git status

git add .

git commit -m "commit message"

git push origin <branch-name>:<branch-name>
```

On git create a PR againt master and merge into master.
This should trigger a build in jenkins


#### Run ansible
```bash
cd ~ && vi .ansible.cfg                                                             #edit ansible config
```
paste the following

```bash
[defaults]
host_key_checking = False
```

```bash
ansible-playbook -i \
/var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/inventory/dev.yml \
/var/lib/jenkins/jobs/ansible/builds/<build-number>/archive/playbooks/common.yml
```
output
![](https://soms-public-assets.s3.amazonaws.com/p11-ansible-result.png)

