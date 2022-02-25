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
- Then the `available` tab
- Search for `EC2`, `Ansible`, `YAML`
- Select and `install plugins without restart`

Installing ansible and configuring ansible on Master instance.
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


Enter python
- `sudo apt-add-repository ppa:ansible/ansible`
- `sudo apt-get install ansible -y`
- `cd /etc/ansible`
- `sudo mkdir group_vars`
- `sudo mkdir group_vars/all`
- `cd group_vars/all`
- `sudo ansible-vault create pass.yml` Add credentials
- `sudo chown jenkins:jenkins pass.yml`

