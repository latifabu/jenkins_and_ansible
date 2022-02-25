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
- Open browser, copy instance IP and add `port 8080`
  
  ![Jenkins password](https://user-images.githubusercontent.com/98215575/155801661-f6d5b10b-209a-4dd5-a4e0-cae312a00d8d.png)
