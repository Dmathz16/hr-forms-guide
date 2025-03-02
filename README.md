# Installing HR Forms on an Ubuntu EC2 Instance – Step-by-Step Guide

## Steps
1. [Prerequisites](#prerequisites)
2. [Connect to EC2 Instance](#connect-to-ec2-instance) 
3. [Update System Packages](#update-system-packages)
4. [Install Nginx](#install-nginx)
5. [Change Root Password](#change-root-password)
6. [Create a New Ubuntu User](#create-a-new-ubuntu-user)
7. [Remove Ubuntu Default User](#remove-ubuntu-default-user) 
8. [Transfer and Extract HR Forms Source Code and Database](#transfer-and-extract-hr-forms-source-code-and-database) 
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

## Transfer and Extract HR Forms Source Code and Database
On your local computer, open cmd/terminal then navigate to the **.pem** file location:
```cmd
cd <PATH_TO_PEM_FILE>
```
Transfer the compressed file
```cmd
scp -i <PATH_TO_PEM_FILE> <PATH_TO_THE_COMPRESSED_SOURCECODE_AND_DATABASE> <NEW_USER>@<EC2_PUBLIC_IP>:/var/www/
```
Verify the transfer
```cmd
ssh -i <PEM_FILE> <NEW_USER>@<EC2_PUBLIC_IP>
ls -lh /var/www/
```
If the compressed file is present, install unrar:
```cmd
sudo apt install unrar -y
```
Extract compressed file to current directory:
```cmd
unrar x <PATH_TO_THE_COMPRESSED_SOURCECODE_AND_DATABASE>
```
Verify extraction:
```cmd
ls -lh /var/www/
```
If extracted, remove compressed file:
```cmd
sudo rm -f <PATH_TO_THE_COMPRESSED_SOURCECODE_AND_DATABASE>
```

## Install MySQL and Set Up Database
Install and set up the MySQL Server:
```cmd
sudo apt-get install mysql-server -y
sudo mysql_secure_installation
```
Connect to the MySQL Server:
```cmd
sudo mysql -u root -p
```
Create a database:
```cmd
CREATE DATABASE <DATABASE_NAME>;
SHOW DATABASES;
```
Configure a new MySQL user account for local connections:
```cmd
CREATE USER '<MYSQL_USERNAME>'@'localhost' IDENTIFIED BY '<MYSQL_PASSWORD>';
GRANT ALL PRIVILEGES ON <DATABASE_NAME>.* TO '<MYSQL_USERNAME>'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Import an existing SQL file using the new MySQL account:
```cmd
sudo mysql -u <MYSQL_USERNAME> -p <DATABASE_NAME> < <PATH_TO_SQL_FILE>
```
Verify the data import:
```cmd
sudo mysql -u <MYSQL_USERNAME> -p
```
```cmd
SHOW DATABASES;
USE <DATABASE_NAME>;
SHOW TABLES;
EXIT;
```
Open the MySQL configuration file:
```cmd
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
Add the following settings at the bottom of the configuration file, then save:
```cmd
[mysqld]
thread_cache_size = 4
innodb_thread_concurrency = 4
innodb_read_io_threads = 2
innodb_write_io_threads = 2
wait_timeout = 300
interactive_timeout = 300
innodb_buffer_pool_size = 128M
event_scheduler = OFF
key_buffer = 8M
performance_schema = 0
```

## Install Python 3, pip, and virtualenv

## Install Dependencies and Set Up PERA Forms Software

## Set Up Gunicorn

## Configure Nginx

## Open Necessary Ports in EC2 Security Group

## Configure UFW

## Test the Application

## Enable SSL with GoDaddy (Optional)

