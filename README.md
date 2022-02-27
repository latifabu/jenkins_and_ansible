# Jenkins and Ansible

1) Create master Jenkins instance on AWS
  - Select create instance
  - Select your VPC
  - Enable subnet
  - Storage - T2 micro
  - Create security groups which allow:
  
  <img width="354" alt="image" src="https://user-images.githubusercontent.com/98215575/155800123-cdab26f8-80a0-4292-8519-67c6afb94a68.png">

- ssh into the instance
- Install master jenkins dependencies:
-  `sudo apt-get update -y && sudo apt-get upgrade -y`
-  `sudo apt install software-properties-common -y`
-  `sudo add-apt-repository ppa:deadsnakes/ppa`
-  `sudo apt install openjdk-8-jre -y`
-  `wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`
-  `sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`
-  `sudo apt-get update -y`
-  `sudo apt-get install jenkins -y`
-  `sudo systemctl start jenkins`
Setting up Jenkins
Open browser, copy instance IP and add `port 8080`
  
  ![Jenkins password](https://user-images.githubusercontent.com/98215575/155801661-f6d5b10b-209a-4dd5-a4e0-cae312a00d8d.png)

- Enter `sudo cat /var/lib/jenkins/secrets/initialAdminPassword in your terminal to obtain the password.`
- Enter the password into the terminal
- Return to browser and see `Customize Jenkins`
- Choose what option suits you best, 
- Either `install suggested plugins` or `select plugins to install`
- Select plugins to install is a much faster opttion.
- Select plugins: SSH Agent, NodeJS, Git Parameter, GitHub, SSH
- Once done
- You will see `create first admin user`
- Enter a username, password, name and e-mail
- Once done select `manage jenkins`
- `manage plugins`

<img width="966" alt="image" src="https://user-images.githubusercontent.com/98215575/155878712-f275c6fa-db90-4fab-afc7-9a5a4fb1f43f.png">


- Then the `available` tab

<img width="675" alt="image" src="https://user-images.githubusercontent.com/98215575/155878731-71a6371c-44c1-4884-a22e-4a46c20db3b9.png">

- Search for `EC2`, `Ansible`, `YAML`
- Select and `install plugins without restart`
- Return to manage Jenkins and select `Global Tool Configuration`

<img width="956" alt="image" src="https://user-images.githubusercontent.com/98215575/155878769-13e12a3d-4682-4703-8df3-bdc34211d8a2.png">

- Then scroll and enter

<img width="685" alt="image" src="https://user-images.githubusercontent.com/98215575/155878838-45446668-9f1b-479e-8d19-f952adcfa952.png">



Installing ansible and configuring ansible on Master instance.
- Return to terminal
- Enter `sudo apt install python3.9 -y`
- `sudo su` - to run as root
- `update-alternatives --install /usr/bin/python python /usr/bin/python3 1`
- `python --version` should show python 3.6.9
- `exit` 
- `sudo apt-get install python3-pip -y`
- `sudo pip3 install awscli boto3 boto3`

Alternative method to install boto:
- `sudo su jenkins`
- `pip3 show boto`
- `pip3 install boto`
- enter `python`
- then after `>>>` enter `import boto`
Example:

<img width="373" alt="image" src="https://user-images.githubusercontent.com/98215575/155805035-b4fff6e0-6e8a-45b0-bb47-b44b351d59fa.png">


Enter python
- `sudo apt-add-repository ppa:ansible/ansible`
- `sudo apt-get install ansible -y`
- `cd /etc/ansible`
- `sudo mkdir group_vars`
- `sudo mkdir group_vars/all`
- `cd group_vars/all`
- `sudo ansible-vault create pass.yml` Add credentials
- `sudo chown jenkins:jenkins pass.yml` - this will give jenkins permission to access to pass.yml
- `ansible-galaxy collection install amazon.aws`
- `scp -i "~/.ssh/keyname.pem" -r app/ ubuntu@ipaddress:~ ` to copy app from an instance
- or `git clone <github link which contains app>`
- `sudo su jenkins` 
- `cd ~`
- `mkdir .ssh` if `.ssh` is not present
- `cd .ssh`
- `ssh-keygen -t rsa -b 4096` generate keys, be mindful of the key names. The names will be used in the ansible playbooks to launch instances on AWS.
- `exit` or `ctrl+d`
- `cd /etc/ansible` 
- create ec2_instance with:
- `sudo nano <name>>.yml`



Two changes were made to our usual playbook to make it run on jenkins.
Under `vars`
- the `ansible_python_interpreter: /usr/bin/python3` was added so python3 acted as an interpreter for ansible
- And the last line: `tags: ['never', 'create_ec2']` without this line the instance will not launch
`
```
---
- hosts: localhost
  connection: local
  gather_facts: yes
  vars_files:
  - /etc/ansible/group_vars/all/pass.yml
  vars:
    ansible_python_interpreter: /usr/bin/python3
    ec2_instance_name: <instance_name> # SECURITY GROUP 
    ec2_sg_name: <security group name> # from corresponding vpc
    ec2_pem_name: <private.key>
  tasks:
  - ec2_key:
      name: "{{ec2_pem_name}}"
      key_material: "{{ lookup('file', '~/.ssh/<key.pub>') }}"
      region: "eu-west-1"
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
  - ec2:
      aws_access_key: "{{aws_access_key}}"
      aws_secret_key: "{{aws_secret_key}}"
      key_name: "{{ec2_pem_name}}"
      instance_type: t2.micro
      image: ami-07d8796a2b0f8d29c
      wait: yes
      group: "{{ec2_sg_name}}"
      region: "eu-west-1"
      count: 1
      vpc_subnet_id: subnet-xxxxxxxx
      assign_public_ip: yes
      instance_tags:
        Name: "{{ec2_instance_name}}"
  tags: ['never', 'create_ec2']

``` 
Now return to Jenkins and create a job that launches the playbook:

- Scroll to and enter
- Playbook path to yml file

![image](https://user-images.githubusercontent.com/98215575/155879112-37706bb0-8060-4639-8bbb-02f4e57bc388.png)

- Under `Vault Credentials` select `add`
- Select `secret text`
- Enter vault password in the `secret field`
- add a description

![image](https://user-images.githubusercontent.com/98215575/155879163-4c1e1671-56ba-4677-8dac-9f51eaff183f.png)

- Scroll to `advanced`

<img width="654" alt="image" src="https://user-images.githubusercontent.com/98215575/155879316-6c5b0042-543e-4baf-8f6a-49998a0f0ca5.png">

- Enter the following

![image](https://user-images.githubusercontent.com/98215575/155879340-8e49c595-f251-42d7-b401-be6301746c27.png)

- Appy and save
- Build the job

Once on AWS copy ip of instances and add to the to hosts file
```
[app]
34.xxx.xxx.xxx ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/<private_key>

[db]
18.xxx.xxx.xxx ssh_connection=ssh ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/<private_key>

```
Test connection with ping
- To ping you must have jenkins as the root

![image](https://user-images.githubusercontent.com/98215575/155879424-a546bfb3-989d-4c55-89ba-fc85dd571122.png)

- Create two more jobs 
- One to run the playbook to set up the app 
- Another to run the playbook that sets up db

Job that will set up app

![image](https://user-images.githubusercontent.com/98215575/155879710-ef029fb4-04e5-48fa-b426-22acc0c21f3a.png)

<img width="695" alt="image" src="https://user-images.githubusercontent.com/98215575/155879740-88eec3f4-fc83-490c-81c2-254a13d721d4.png">

<img width="680" alt="image" src="https://user-images.githubusercontent.com/98215575/155879799-eadd0ec4-f667-430f-9770-eda9d4fea877.png">

- start_app.yml looked like:
```
---
- hosts: app
  gather_facts: yes
  become: yes
  tasks:
  -  name: syncing app folder
     synchronize:
       src: /home/ubuntu/app
       dest: ~/
  -  name: upgrade
     apt: upgrade=yes
  -  name: load a specific version of nodejs
     shell: curl -sl https://deb.nodesource.com/setup_6.x | sudo -E bash -
  -  name: install the required packages
     apt:
       pkg:
         - nginx
         - nodejs
         - npm
       update_cache: yes
  -  name: nginx configuration for reverse proxy
     synchronize:
       src: /home/ubuntu/app/default
       dest: /etc/nginx/sites-available/default
  -  name: nginx restart
     service:
       name: nginx
       state: restarted
  -  name: setting db variable
     lineinfile:
       dest: /home/ubuntu/.bashrc
       line: 'export DB_HOST=mongodb://<db_ip>:27017/posts'

```
- Job that sets up db

![image](https://user-images.githubusercontent.com/98215575/155879879-42d6573c-fcd7-4826-8fc7-7ab9cb2d8cc4.png)

- Like the previous job, select advance

- <img width="664" alt="image" src="https://user-images.githubusercontent.com/98215575/155879901-ac591852-ef92-4b3f-b2b6-effdac296ab2.png">


Run the job

Blockers:
- Jenkins
