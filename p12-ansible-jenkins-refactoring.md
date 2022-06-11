# ANSIBLE & JENKINS REFACTORING

### Project 12

Continuate from [Project 11](/p-11-ansible-config-mgt.md)

Refactoring - is making changes to the source code without changing expected behaviour of the software.

### Jenkins job Enhancement
Our current jenkins job from project 11 creates a seperate directory like `/var/lib/jenkins/jobs/ansible/builds/<build-number>/archive` based on build number and stores artifacts

1. Login to Jenkins-ansible server and create directory and `/`
   1. ```bash
      sudo mkdir -p /artifacts
      sudo chmod -R 777 /artifacts
      mkdir /artifacts/ansible-config-artifact 
    ```

2. Goto Jenkins web console and install jenkins plugin `Copy Artifact`
3. Create a new freestyle project called `save_artifacts`. This job will run after the `Ansible` job from project 11 is executed. And will move the most recent artifact to `/artifacts/ansible-config-artifact `. To setup the `save_artifacts`, go to `General > Discard Old Builds`, `Build Triggers > Build after other projects are built` and `Add Build Step`. The Image explains better
![](https://soms-public-assets.s3.amazonaws.com/images/p12-artifact-mgt-jenkins1.png)
![](https://soms-public-assets.s3.amazonaws.com/images/p12-artifact-mgt-jenkins2.png)
4. After successful setup, update the README.md, commit the change and see the two jobs run
5. List /artifacts/ansible-config-artifact to confirm later build was copied successfully
![](https://soms-public-assets.s3.amazonaws.com/images/p12-list-copied-files.png)



#### Reinstalling jenkins
If you have any issues working with jenkins here due to previous changes then remove and reinstall
```bash
# Remove completely
sudo service jenkins stop
sudo yum remove jenkins
sudo apt-get remove --purge jenkins
sudo apt-get remove --auto-remove jenkins
sudo rm /etc/apt/sources.list.d/jenkins.list
sudo rm /etc/apt/sources.list.d/jenkins.list.save
sudo rm /var/lib/jenkins


# Reinstall
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo service jenkins restart

sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```


### Refactoring Ansible Code
Refactor Ansible code by importing other playbooks
In your `Ansible-config-mgt` project from project 11
1. `git pull` from master
2. create a new branch `refactor`
3. create a new file `playbook/site.yml`
4. create new folder `static-assignments`
5. move `playbook/common.yml` into `static-assignments/`
6. create a new file `static-assignments/common-del.yml`
   
Add the following content in

1. `playbook/site.yml`
```yml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
- import_playbook: ../static-assignments/common-del.yml
```

2. `static-assignments/common-del.yml`
This will delete the wireshart already install in all the servers
```yml
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

Goto the server to test

1. ssh into the `ansible-jenkins` server
2. `git clone https://github.com/chis0m/Ansible-config-mgt.git`
3. checkout into `refactor` branch
4. Run playbook `ansible-playbook -i inventory/dev.yml playbooks/site.yml`
5. ssh into those servers and run `wireshark --version` to confirm if still installed



### Using Roles in Ansible
1. Launch 2 Rhel8 EC2 instances with name `Web1-UAT ` and `Web1-UAT `
2. In your `Ansible-config-mgt` project checkout into a new branch called `roles` from the `refactor` branch - so you can have previous changes
3. create the following manually `roles/webserver ...`
```
roles
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    └── tasks
        └── main.yml
```

use `ansible-galaxy init webserver` to create the webserver directory if you have ansible installed locally

3. In `inventory/uat.yml` paste the following
```yml
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
```
4. In `roles/webserver/tasks/main.yml` paste the following
```yml
---
- name: install apache                                                      # configure apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git                                                         # install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo                                                        # clone repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/chis0m/tooling.git                             # remmember to subsitute chis0m with your git name
    dest: /var/www/html
    force: yes

- name: copy html content to one level up                                   # copy html to /var/www/html
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started                                 # start httpd server
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory                    # remove /var/www/html/html
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

5. ssh into ansible-jenkins server, update the `~/.ansible.cfg` file we created in project 11; point ansible to a different role path
 `vi ~/.ansible.cfg`

paste the following

```bash
[defaults]
host_key_checking = False
roles_path = /artifacts/ansible-config-artifact/roles
```

6. create `static-assignments/uat-webservers.yml` and paste the following to **reference the webserver role**
```yml
---
- hosts: uat-webservers
  roles:
     - webserver
```

7. Update the `playbook/site.yml` to import the playbook `uat-webservers.yml`
```yml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
- import_playbook: ../static-assignments/common-del.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

8. Push changes to git and create a PR against master
```bash
git add .
git commit -m 'create role'
git push origin -u roles
```
After merging the PR against master

9. Goto `/artifacts/ansible-config-artifact` in the ansible-jenkins server to confirm jenkins build and copy was successful

10. Run `ansible-playbook -i /artifacts/ansible-config-artifact/inventory/uat.yml /artifacts/ansible-config-artifact/playbooks/site.yml`
    
You should get something like this
![](https://soms-public-assets.s3.amazonaws.com/images/p12-result.png)    