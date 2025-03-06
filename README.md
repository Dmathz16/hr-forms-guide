# Installing HR Forms on an Ubuntu EC2 Instance – Step-by-Step Guide

## Steps
1. [Prerequisites](#prerequisites)
2. [Open Necessary Ports in EC2 Security Group](#open-necessary-ports-in-ec2-security-group) 
3. [Connect to EC2 Instance](#connect-to-ec2-instance) 
4. [Update System Packages](#update-system-packages)
5. [Install Nginx](#install-nginx)
6. [Change Root Password](#change-root-password)
7. [Create a New Ubuntu User](#create-a-new-ubuntu-user)
8. [Remove Ubuntu Default User](#remove-ubuntu-default-user) 
9. [Transfer and Extract HR Forms Source Code and Database](#transfer-and-extract-hr-forms-source-code-and-database) 
10. [Install MySQL and Set Up Database](#install-mysql-and-set-up-database)  
11. [Install Python 3 and Its Dependencies](#install-python-3-and-its-dependencies) 
12. [Install Dependencies and Set Up PERA Forms Software](#install-dependencies-and-set-up-pera-forms-software) 
13. [Set Up Gunicorn and WSGI](#set-up-gunicorn-and-wsgi)
14. [Run Flask App as a System Service](#run-flask-app-as-a-system-service) 
15. [Configure Nginx](#configure-nginx) 
16. [Configure UFW](#configure-ufw) 
17. [Test the Application](#test-the-application) 
18. [Enable SSL (Optional)](#enable-ssl-optional)
19. [Guide Placeholders](#guide-placeholders)

## Prerequisites
Before starting, ensure you have:
* An AWS EC2 instance running Ubuntu
* A domain name (optional, but recommended)
* The HR Forms source code and database dump
* Your EC2 key pair (.pem file) for SSH access
* Basic knowledge of Linux commands

## Open Necessary Ports in EC2 Security Group
1. In the AWS Console, go to EC2 > Instances, find your instance, and click its Security Group.
2. Under the Inbound rules tab, click Edit inbound rules.
3. Click Add Rule and open these ports:
	* 22 (SSH) → My IP
	* 80 (HTTP), 443 (HTTPS), 5000 (Flask, optional) → Anywhere (0.0.0.0/0)
4. Click Save rules. 

## Connect to EC2 Instance
* Use SSH to connect to your instance. Replace <PATH_TO_PEM_FILE> and <EC2_PUBLIC_IP> accordingly: 
```cmd
ssh -i <PATH_TO_PEM_FILE> <EXISTING_USER>@<EC2_PUBLIC_IP>
```

## Update System Packages
* Ensure the system is up to date:
```cmd
sudo apt update && sudo apt upgrade -y
sudo timedatectl set-timezone Asia/Manila
timedatectl
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
* For security reasons, it's recommended to change the root password:
	```cmd
	sudo passwd root
	```
	You'll be prompted to enter and confirm a **new root password**. 

## Create a New Ubuntu User
* To create a new user, switch to the root user:
	```cmd
	sudo -i
	```
* Then, add the new user:
	```cmd
	adduser <NEW_USER>
	```

* Add the user to the sudo group:
	```cmd
	usermod -aG sudo <NEW_USER>
	```

* Add the user to the www-data group and set the appropriate permissions for /var/www:
	```cmd
	usermod -aG www-data <NEW_USER>
	```
* Set correct permissions and ownership:
	```cmd
	sudo chmod -R 775 /var/www
	sudo chown -R <NEW_USER>:www-data /var/www
	```

* Open the SSH configuration file:
	```cmd
	nano /etc/ssh/sshd_config
	```
* Uncomment or modify the following line:
	```cmd
	PasswordAuthentication yes
	```
* Save the file and restart the SSH service:
	```cmd
	sudo systemctl restart ssh.service
	```

* Create the .ssh directory for the new user:
	```cmd
	sudo mkdir -p /home/<NEW_USER>/.ssh
	sudo chmod 700 /home/<NEW_USER>/.ssh
	```
* Copy the authorized SSH keys from the existing user (ubuntu):
	```cmd
	sudo cp /home/<EXISTING_USER>/.ssh/authorized_keys /home/<NEW_USER>/.ssh/authorized_keys
	```
* Set correct permissions and ownership:
	```cmd
	sudo chmod 600 /home/<NEW_USER>/.ssh/authorized_keys
	sudo chown -R <NEW_USER>:<NEW_USER> /home/<NEW_USER>/.ssh
	```

## Remove Ubuntu Default User
* Exit the console and then SSH into the new user:
	```cmd
	ssh -i <PATH_TO_PEM_FILE> <NEW_USER>@<EC2_PUBLIC_IP>
	```
* Switch to the root user:
	```cmd
	sudo -i
	```
* List processes running under the ubuntu user:
	```cmd
	ps -u <EXISTING_USER>
	```
* Terminate processes one by one using their Process ID (PROCESS_ID): 
	```cmd
	sudo kill <PROCESS_ID>
	```
* Remove the ubuntu user after stopping all processes: 
	```cmd
	sudo userdel -r <EXISTING_USER>
	```
* Exit the root session: 
	```cmd
	exit;
	```
* Exit the console: 
	```cmd
	exit;
	```

## Transfer and Extract HR Forms Source Code and Database
* On your local computer, open cmd/terminal then navigate to the **.pem** file location:
	```cmd
	cd <PATH_TO_PEM_FILE>
	```
* Transfer the compressed file
	```cmd
	scp -i <PATH_TO_PEM_FILE> <PATH_TO_THE_COMPRESSED_SOURCECODE> <NEW_USER>@<EC2_PUBLIC_IP>:/var/www/
	```
* Verify the transfer
	```cmd
	ssh -i <PATH_TO_PEM_FILE> <NEW_USER>@<EC2_PUBLIC_IP>
	ls -lh /var/www/
	```
* If the compressed file is present, install unrar:
	```cmd
	sudo apt install unrar -y
	```
* Extract compressed file to current directory:
	```cmd
 	cd /var/www
	unrar x <PATH_TO_THE_COMPRESSED_SOURCECODE>
	```
* Verify extraction:
	```cmd
	ls -lh /var/www/
	```
* If extracted, remove compressed file:
	```cmd
	sudo rm -f <PATH_TO_THE_COMPRESSED_SOURCECODE>
	```

## Install MySQL and Set Up Database
* Install and set up the MySQL Server:
	```cmd
	sudo apt-get install mysql-server -y
	sudo mysql_secure_installation
	```
* Connect to the MySQL Server:
	```cmd
	sudo mysql -u root -p
	```
* Create a database:
	```cmd
	CREATE DATABASE <DATABASE_NAME>;
	SHOW DATABASES;
	```
* Configure a new MySQL user account for local connections:
	```cmd
	CREATE USER '<MYSQL_USERNAME>'@'localhost' IDENTIFIED BY '<MYSQL_PASSWORD>';
	GRANT ALL PRIVILEGES ON <DATABASE_NAME>.* TO '<MYSQL_USERNAME>'@'localhost';
	FLUSH PRIVILEGES;
	EXIT;
	```
* Import an existing SQL file using the new MySQL account:
	```cmd
	sudo mysql -u <MYSQL_USERNAME> -p <DATABASE_NAME> < <PATH_TO_SQL_FILE>
	```
* Verify the data import:
	```cmd
	sudo mysql -u <MYSQL_USERNAME> -p
	```
	```cmd
	SHOW DATABASES;
	USE <DATABASE_NAME>;
	SHOW TABLES;
	EXIT;
	```
* Open the MySQL configuration file:
	```cmd
	sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
	```
* Add the following settings at the bottom of the configuration file, then save:
	```cmd
	thread_cache_size = 4
	innodb_thread_concurrency = 4
	innodb_read_io_threads = 2
	innodb_write_io_threads = 2
	wait_timeout = 300
	interactive_timeout = 300
	innodb_buffer_pool_size = 128M
	event_scheduler = OFF
	performance_schema = 0
	```
 * Restart MySQL Server:
	```cmd
	sudo systemctl restart mysql
	```

## Install Python 3 and Its Dependencies
```cmd
sudo apt install python3 python3-pip python3-venv -y
```

## Install Dependencies and Set Up PERA Forms Software
* Navigate to the source code path:
	```cmd
	cd <PATH_TO_THE_UNCOMPRESSED_SOURCECODE>
	```
* Install the virtual environment:
	```cmd
	python3 -m venv .venv
	```
* Activate the virtual environment:
	```cmd
	. .venv/bin/activate
	```
* Install app dependencies:
	```cmd
	pip3 install -r requirements.txt
	```
* Open app configurations:
	```cmd
	nano application/__init__.py
	```
* Update the database variable values, then save:
	```cmd
	DB_SERVER   = "localhost"
	DB_PORT     = "3306"
	DB_USERNAME = "<MYSQL_USERNAME>"
	DB_PASSWORD = "<MYSQL_PASSWORD>"
	DB_NAME     = "<DATABASE_NAME>"
	```
* Run the app:
	```cmd
	flask --app application run --debug --host 0.0.0.0
	```
	Open the app in a browser using the URL **http://<EC2_PUBLIC_IP>:5000** to check if it is running

## Set Up Gunicorn and WSGI
If the app still running press **CTRL + C** to stop.
* Install gunicorn:
	```cmd
	pip3 install gunicorn
	```
* Run app with gunicorn:
	```cmd
	gunicorn --bind 0.0.0.0:5000 application:app
	```
	Open the app in a browser using the URL **http://<EC2_PUBLIC_IP>:5000** to check if it is running

* Stop gunicorn by pressing **CTRL + C**, then create wsgi.py file inside project directory:
	```cmd
	sudo nano <PATH_TO_THE_UNCOMPRESSED_SOURCECODE>/wsgi.py
	```
 * Paste this inside wsgi.py file and save:
	```cmd
	from application import app
	
	if __name__ == '__main__':
		app.run(host='0.0.0.0')
	```
* Run app with gunicorn + wsgi:
	```cmd
	gunicorn --bind 0.0.0.0:5000 wsgi:app
	```
	Open the app in a browser using the URL **http://<EC2_PUBLIC_IP>:5000** to check if it is running

	**If it isn't running**, check the wsgi.py code for spacing/indent problem then save and run it again.

* Press **CTRL + C** to stop gunicorn, then deactivate environment if done testing:
	```cmd
	deactivate
	```
 
## Run Flask App as a System Service
* After deactivating the Python environment, create a new service file:
	```cmd
	sudo nano /etc/systemd/system/hr_forms.service
	```
* Paste the following code into the service file, update the values as needed, then save:
	```cmd
	[Unit]
	Description=PERAMPC HR Forms
	After=network.target
	
	[Service]
	User=<NEW_USER> 
	Group=www-data
	WorkingDirectory=<PATH_TO_THE_UNCOMPRESSED_SOURCECODE>
	Environment="PATH=<PATH_TO_THE_UNCOMPRESSED_SOURCECODE>/.venv/bin"
	ExecStart=<PATH_TO_THE_UNCOMPRESSED_SOURCECODE>/.venv/bin/gunicorn --workers 2 --timeout 20 --bind unix:application.sock -m 007 wsgi:app
	
	# Log output configuration
	StandardOutput=append:/var/log/hr_forms/gunicorn_access.log
	StandardError=append:/var/log/hr_forms/gunicorn_error.log
	
	[Install]
	WantedBy=multi-user.target
	```
* Create a directory for application logs:
	```cmd
	sudo mkdir -p /var/log/hr_forms
	```
* Set permissions for the log directory:
	```cmd
	sudo chown www-data:www-data /var/log/hr_forms
	```
* Start and enable the new service:
	```cmd
	sudo systemctl daemon-reload
	sudo systemctl start hr_forms
	sudo systemctl enable hr_forms
	sudo systemctl status hr_forms
	```

## Configure Nginx
If hr_forms service status is still showing close it by pressing **CTRL + C**
* Create an Nginx configuration file for the application:
	```cmd
	sudo nano /etc/nginx/sites-available/hr_forms.conf
	```
* Add the following configuration to handle domain redirection and proxy requests:
	```cmd
	server {
	    listen 80;
	    server_name www.<DOMAIN_NAME>;
	    return 301 http://<DOMAIN_NAME>$request_uri;
	}
	server {
	    listen 80;
	    server_name <EC2_PUBLIC_IP>;  
	    return 301 http://<DOMAIN_NAME>$request_uri;
	}
	server {
	    listen 80;
	    server_name <DOMAIN_NAME>;
	    location / {
	        include proxy_params;
	        proxy_pass http://unix:<PATH_TO_THE_UNCOMPRESSED_SOURCECODE>/application.sock;
	    }
	}
	```
* Enable the new configuration by creating a symbolic link:
	```cmd
	sudo ln -s /etc/nginx/sites-available/hr_forms.conf /etc/nginx/sites-enabled/
	```
* Check for syntax errors in the Nginx configuration:
	```cmd
	sudo nginx -t
	```
* Restart Nginx to apply the changes:
	```cmd
	sudo systemctl restart nginx
	```

## Configure UFW
```cmd
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw allow 3306/tcp
sudo ufw enable
sudo ufw status
```

## Test the Application
Open the app in a browser using the URL **http://<EC2_PUBLIC_IP>** or Domain **http://<DOMAIN_NAME>** to check if it is running.

## Enable SSL (Optional)
* Install OpenSSL:
```cmd
sudo apt install openssl
```
* Create a directory for certificate files: 
```cmd
sudo mkdir -p /etc/ssl/certs/hr_forms
```
* Navigate to the directory: 
```cmd
cd /etc/ssl/certs/hr_forms
```
* Generate a private key without a passphrase:
```cmd
sudo openssl genpkey -algorithm RSA -out dmathz.com.key
```
  Note: **Including pass phrase increases security**
* Generate a Certificate Signing Request (CSR):
```cmd
sudo openssl req -new -key dmathz.com.key -out dmathz.com.csr
```
* Open the CSR file:
```cmd
sudo nano dmathz.com.csr
```
  **Copy the texts in .csr, it will be used to generate ssl from ssl providers (e.g. GoDaddy)** then **CTRL + X** to close the file
* **Exit the SSH session**, then in your browser, **go to the GoDaddy website** to **generate and download the SSL certificate**
* Transfer all required files from your computer to the Ubuntu server:
```cmd
scp -i <PATH_TO_PEM_FILE> <PATH_TO_SSL_CERT_FILE> <NEW_USER>@<EC2_PUBLIC_IP>:/var/www
```
* **SSH into the server** and move the SSL certificate files to the certificate directory:
```cmd
sudo mv <PATH_TO_SSL_CERT_FILE> /etc/ssl/certs/hr_forms/
```
* Set the correct file permissions:
```cmd
sudo chmod 600 /etc/ssl/certs/hr_forms/*
```
* Verify that the files exist in the directory:
```cmd
cd /etc/ssl/certs/hr_forms
ls
```
* Open the HR Forms service configuration file:
```cmd
sudo nano /etc/nginx/sites-available/hr_forms.conf
```
* Replace all text with the following configuration:
```cmd
server {
    listen 80;
    server_name www.<DOMAIN_NAME> <DOMAIN_NAME> <EC2_PUBLIC_IP>;
    return 301 https://<DOMAIN_NAME>$request_uri;
}

server {
    listen 443 ssl;
    server_name www.<DOMAIN_NAME> <EC2_PUBLIC_IP>;

    ssl_certificate /etc/ssl/certs/hr_forms/<CERT_NAME>.crt;
    ssl_certificate_key /etc/ssl/certs/hr_forms/<PRIVATE_KEY>.key;
    ssl_trusted_certificate /etc/ssl/certs/hr_forms/<BUNDLE_NAME>.crt;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    return 301 https://<DOMAIN_NAME>$request_uri;
}

server {
    listen 443 ssl;
    server_name <DOMAIN_NAME>;

    ssl_certificate /etc/ssl/certs/hr_forms/<CERT_NAME>.crt;
    ssl_certificate_key /etc/ssl/certs/hr_forms/<PRIVATE_KEY>.key;
    ssl_trusted_certificate /etc/ssl/certs/hr_forms/<BUNDLE_NAME>.crt;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/hr_forms/application.sock;
    }
}
```
* Check the Nginx configuration for errors:
```cmd
sudo nginx -t
```
* Reload Nginx to apply the changes:
```cmd
sudo systemctl reload nginx
```

## Guide Placeholders
**Ubuntu User:**
* <NEW_USER> – A newly created Ubuntu user for security purposes.
* <EXISTING_USER> – The default Ubuntu user, typically named **ubuntu**.

**Paths:**
* <PATH_TO_PEM_FILE> – The file path to the SSH .pem key.
* <PATH_TO_SSL_CERT_FILE> – The file path to the SSL certificate files.
* <PATH_TO_THE_COMPRESSED_SOURCECODE> – The file path to the compressed source code archive.
* <PATH_TO_THE_UNCOMPRESSED_SOURCECODE> – The file path to the extracted/uncompressed source code.
* <PATH_TO_SQL_FILE> – The file path to the database SQL file inside the uncompressed source code directory.

**MySQL:**
* <MYSQL_USERNAME> – The MySQL username credential.
* <MYSQL_PASSWORD> – The MySQL password credential.
* <DATABASE_NAME> – The name of the database used by the application.

**URL:**
* <EC2_PUBLIC_IP> – The public IP address of the AWS EC2 instance.
* <DOMAIN_NAME> – The domain name associated with the application.

**SSL:**
* <CERT_NAME> – The name of the SSL certificate.
* <PRIVATE_KEY> – The private key for the SSL certificate.
* <BUNDLE_NAME> – The SSL bundle file (CA bundle).
**Others:**
* <PROCESS_ID> – The process ID of a running application on Ubuntu.

