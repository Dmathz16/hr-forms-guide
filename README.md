# Installing HR Forms on an Ubuntu EC2 Instance – Step-by-Step Guide

## Contents
1. [Prerequisites](#prerequisites)
2. [Connect to EC2 Instance](#) 
3. [Update System Packages](#) 
4. [Create a New Ubuntu User](#) 
5. [Transfer HR Forms Source Code and Database](#) 
6. [Install Nginx](#)  
7. [Install MySQL and Set Up Database](#)  
8. [Install Python 3, pip, and virtualenv](#) 
9. [Install Dependencies and Set Up PERA Forms Software](#) 
10. [Set Up Gunicorn](#)  
11. [Configure Nginx](#) 
12. [Open Necessary Ports in EC2 Security Group](#) 
13. [Configure UFW](#) 
14. [Test the Application](#) 
15. [Enable SSL with GoDaddy](#) 

## Prerequisites
Before starting, ensure you have:
  * An AWS EC2 instance running Ubuntu
  * A domain name (optional, but recommended)
  * SSH access to your EC2 instance
  * Basic knowledge of Linux commands
