# Installing HR Forms on an Ubuntu EC2 Instance – Step-by-Step Guide

## Steps
1. [Prerequisites](#prerequisites)
2. [Connect to EC2 Instance](#connect-to-ec2-instance) 
3. [Update System Packages](#update-system-packages)
4. [Install Nginx](#install-nginx)
5. [Change Root Password](#change-root-password)
6. [Create a New Ubuntu User](#create-a-new-ubuntu-user)
7. [Remove Ubuntu Default User](#remove-ubuntu-default-user) 
8. [Transfer HR Forms Source Code and Database](#transfer-hr-forms-source-code-and-database) 
9. [Install MySQL and Set Up Database](#install-mysql-and-set-up-database)  
10. [Install Python 3, pip, and virtualenv](#install-python-3-pip-and-virtualenv) 
11. [Install Dependencies and Set Up PERA Forms Software](#install-dependencies-and-set-up-pera-forms-software) 
12. [Set Up Gunicorn](#set-up-gunicorn)  
13. [Configure Nginx](#configure-nginx) 
14. [Open Necessary Ports in EC2 Security Group](#open-necessary-ports-in-ec2-security-group) 
15. [Configure UFW](#configure-ufw) 
16. [Test the Application](#test-the-application) 
17. [Enable SSL with GoDaddy (Optional)](#enable-ssl-with-goDaddy-optional) 

## Prerequisites
Before starting, ensure you have:
  * An AWS EC2 instance running Ubuntu
  * A domain name (optional, but recommended)
  * The HR Forms source code and database dump
  * Your EC2 key pair (.pem file) for SSH access
  * Basic knowledge of Linux commands

## Connect to EC2 Instance
Use SSH to connect to your instance. Replace <PEM_FILE> and <EC2_PUBLIC_IP> accordingly.
```cmd
ssh -i <PEM_FILE> <EXISTING_USER>@<EC2_PUBLIC_IP>
```

## Update System Packages
Ensure the system is up to date:
```cmd
sudo apt update && sudo apt upgrade -y
```

## Install Nginx 
```cmd
sudo apt install nginx -y
```
```cmd
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Change Root Password
For security reasons, it's recommended to change the root password:
```cmd
sudo passwd root
```
You'll be prompted to enter and confirm a **new root password**. 

## Create a New Ubuntu User
To create a new user, switch to the root user:
```cmd
sudo -i
```
Then, add the new user:
```cmd
adduser <NEW_USER>
```

Add the user to the sudo group:
```cmd
usermod -aG sudo <NEW_USER>
```

Add the user to the www-data group and set the appropriate permissions for /var/www:
```cmd
usermod -aG www-data <NEW_USER>
```
Set correct permissions and ownership:
```cmd
sudo chmod -R 775 /var/www
sudo chown -R <NEW_USER>:www-data /var/www
```

Open the SSH configuration file:
```cmd
nano /etc/ssh/sshd_config
```
Uncomment or modify the following line:
```cmd
PasswordAuthentication yes
```
Save the file and restart the SSH service:
```cmd
sudo systemctl restart ssh.service
```

Create the .ssh directory for the new user:
```cmd
sudo mkdir -p /home/<NEW_USER>/.ssh
sudo chmod 700 /home/<NEW_USER>/.ssh
```
Copy the authorized SSH keys from the existing user (ubuntu):
```cmd
sudo cp /home/<EXISTING_USER>/.ssh/authorized_keys /home/<NEW_USER>/.ssh/authorized_keys
```
Set correct permissions and ownership:
```cmd
sudo chmod 600 /home/<NEW_USER>/.ssh/authorized_keys
sudo chown -R <NEW_USER>:<NEW_USER> /home/<NEW_USER>/.ssh
```

## Remove Ubuntu Default User
Exit the console and then SSH into the new user:
```cmd
ssh -i <PEM_FILE> <NEW_USER>@<EC2_PUBLIC_IP>
```
Switch to the root user:
```cmd
sudo -i
```
List processes running under the ubuntu user
```cmd
ps -u <EXISTING_USER>
```
Terminate processes one by one using their Process ID (PROCESS_ID)
```cmd
sudo kill <PROCESS_ID>
```
Remove the ubuntu user after stopping all processes
```cmd
sudo userdel -r <EXISTING_USER>
```
Exit the root session
```cmd
exit
```

## Transfer HR Forms Source Code and Database

## Install MySQL and Set Up Database

## Install Python 3, pip, and virtualenv

## Install Dependencies and Set Up PERA Forms Software

## Set Up Gunicorn

## Configure Nginx

## Open Necessary Ports in EC2 Security Group

## Configure UFW

## Test the Application

## Enable SSL with GoDaddy (Optional)

