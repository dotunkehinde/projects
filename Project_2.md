# WEB STACK IMPLEMENTATION (LEMP STACK)

## Introduction
This project will install the NGINX Webserver. 

## Step 0 - Preparing prerequisites
- Downloaded and installed Git Bash. 
- Launched an instance of Ubuntu Server 20.04 LTS
- Logged in to the AWS EC2 instance with git bash.  
![gitbash login](https://user-images.githubusercontent.com/20668013/120398278-cd4b2580-c331-11eb-85b4-fbcd0d275e61.JPG)

## Step 1 – Installing the Nginx Web Server
- Ran ```sudo apt update``` and ```sudo apt install nginx``` to install nginx.
- Verified that nginx was successfully installed by runnign ```sudo systemctl status nginx```  
![nginx running](https://user-images.githubusercontent.com/20668013/120398548-61b58800-c332-11eb-826a-c5d47825d293.JPG)
- Opened TCP port 80 in security group.
- ```http://18.117.187.158/```  
- ![nginx welc](https://user-images.githubusercontent.com/20668013/120399174-67f83400-c333-11eb-858e-cddd662a2d5e.JPG)

## Step 2 — Installing MySQL
- Installed MySQL using apt ```$ sudo apt install mysql-server```.
- Installed MySQL Secure Installation ```$ sudo mysql_secure_installation```
- Logged in to MySQL  
- ![mysql2](https://user-images.githubusercontent.com/20668013/120399738-87439100-c334-11eb-84c4-b3cd1465296a.JPG)

 ## Step 3 — Installing PHP
- Installed PHP ```$ sudo apt install php-fpm php-mysql ```
- Verified installation of PHP ```$ php -v```  
![php verify](https://user-images.githubusercontent.com/20668013/120052716-7cc88500-c01e-11eb-9c84-6bd0162a594b.JPG)

## Step 4 — Configuring Nginx to Use PHP Processor
- Created root folder for our new domain as follows:
```
sudo mkdir /var/www/projectLEMP
```
- Assigned ownership of the directory with the $USER environment variable
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```
- Opened a new configuration file in Nginx's directory using the nano text editor
```
sudo nano /etc/nginx/sites-available/projectLEMP
```
```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
- Activated configuration by linking to the config file from Nginx Directory
```
$ sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

```
- Tested Nginx configuraton using ``` sudo nginx -t ```
![nginx verify](https://user-images.githubusercontent.com/20668013/120401083-1c478980-c337-11eb-9a81-2b2ef49eb584.JPG)
- Disabled default Nginx host 
``` 
sudo unlink /etc/nginx/sites-enabled/default
```
- Reloaded Nginx to apply changes
```
sudo systemctl reload nginx
```
- Created an index.html file
``` http://18.117.187.158/ ```
![hello lemp](https://user-images.githubusercontent.com/20668013/120401422-d2ab6e80-c337-11eb-9d09-95a870d76024.JPG)

## Step 5 – Testing PHP with Nginx
- Created a php file in document root
```
nano /var/www/projectLEMP/info.php
```
```
<?php
phpinfo();
```
```
http://18.117.187.158/info.php
```
![phpinfo](https://user-images.githubusercontent.com/20668013/120402058-2b2f3b80-c339-11eb-9f4e-5bc11d5de8c5.JPG)
- Removed php configuration file or security reasons.
```
 sudo rm /var/www/your_domain/info.php
```
## Step 6 — Retrieving data from MySQL database with PHP


#### To access this project, please click [here!](ec2-18-191-149-182.us-east-2.compute.amazonaws.com)








 





