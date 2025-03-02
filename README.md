# Installing HR Forms on an Ubuntu EC2 Instance – Step-by-Step Guide

## Contents
1. [Prerequisites](#prerequisites)
2. [Connect to EC2 Instance](#connect-to-ec2-instance) 
3. [Update System Packages](#update-system-packages)
4. [Install Nginx](#install-nginx)
5. [Change Root Password](#change-root-password)
6. [Create a New Ubuntu User](#create-a-new-ubuntu-user) 
7. [Transfer HR Forms Source Code and Database](#transfer-hr-forms-source-code-and-database) 
8. [Install MySQL and Set Up Database](#install-mysql-and-set-up-database)  
9. [Install Python 3, pip, and virtualenv](#install-python-3-pip-and-virtualenv) 
10. [Install Dependencies and Set Up PERA Forms Software](#install-dependencies-and-set-up-pera-forms-software) 
11. [Set Up Gunicorn](#set-up-gunicorn)  
12. [Configure Nginx](#configure-nginx) 
13. [Open Necessary Ports in EC2 Security Group](#open-necessary-ports-in-ec2-security-group) 
14. [Configure UFW](#configure-ufw) 
15. [Test the Application](#test-the-application) 
16. [Enable SSL with GoDaddy (Optional)](#enable-ssl-with-goDaddy-optional) 

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

