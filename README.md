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
ssh -i <PEM_FILE> ubuntu@<EC2_PUBLIC_IP>
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
sudo passwd root
```
Then, add the new user:
```cmd
sudo passwd root
```

Add the user to the sudo group:
```cmd
sudo passwd root
```

Add the user to the www-data group and set the appropriate permissions for /var/www:
```cmd
sudo passwd root
```
Set correct permissions and ownership:
```cmd
sudo passwd root
```

Open the SSH configuration file:
```cmd
sudo passwd root
```
Uncomment or modify the following line:
```cmd
sudo passwd root
```
Save the file and restart the SSH service:
```cmd
sudo passwd root
```

Create the .ssh directory for the new user:
```cmd
sudo passwd root
```
Copy the authorized SSH keys from the existing user:
```cmd
sudo passwd root
```
Set correct permissions and ownership:
```cmd
sudo passwd root
```

## Remove Ubuntu Default User

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

