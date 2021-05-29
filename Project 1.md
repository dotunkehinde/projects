# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

## Introduction
A technology stack is a set of frameworks and tools used to develop a software product. This projects sets up Linux, Apache, MySQL, and PHP on an Ubuntu Instance in AWS.

## Step 0 - Preparing prerequisites
- Opened an Amazon Web Services Account
- Launched an instance of Ubuntu Server 20.04 LTS, located at [Public IPv4 DNS](ec2-18-191-149-182.us-east-2.compute.amazonaws.com)
![AWS - Launch Instance](https://user-images.githubusercontent.com/20668013/120049794-0bcfa000-c013-11eb-9c42-25ab7012f931.JPG)
- Logged in to the AWS EC2 instance with putty.
![putty](https://user-images.githubusercontent.com/20668013/120050737-ea23e800-c015-11eb-9531-cabff39ada8a.JPG)

## Step 1 — Installing Apache and Updating the Firewall
 - Updated my list of packages with ```$ sudo apt update``` command.
  ![apt update](https://user-images.githubusercontent.com/20668013/120051581-1d1bab00-c019-11eb-8745-3a29a2814f8f.JPG)
 - Installed Apache with the ``` $ sudo apt install apache2 ``` command
 - Verified that Apache2 is running using ``` $ sudo systemctl status apache2```
![apache verify](https://user-images.githubusercontent.com/20668013/120051742-bc40a280-c019-11eb-91d9-0e0c4e29a4a6.JPG)
- Edited inbound rules to include http on port 80 with source from anywhere.
- Verified that Apache is accessible through my firewall
![Apache2 Default Page](https://user-images.githubusercontent.com/20668013/120052120-9916f280-c01b-11eb-9c12-1031a36f38f0.JPG)

## Step 2 — Installing MySQL
- Installed MySQL using apt ```$ sudo apt install mysql-server```.
- Installed MySQL Secure Installation ```$ sudo mysql_secure_installation```
- Logged in to MySQL 
 ![mysql](https://user-images.githubusercontent.com/20668013/120052516-9d440f80-c01d-11eb-8f73-f72f5c5368af.JPG)
 
 ## Step 3 — Installing PHP
- Installed PHP ```$ sudo apt install php libapache2-mod-php php-mysql```
- Verified installation of PHP ```$ php -v```
![php verify](https://user-images.githubusercontent.com/20668013/120052716-7cc88500-c01e-11eb-9c84-6bd0162a594b.JPG)

## Step 4 — Creating a Virtual Host for your Website using Apache
- Ran ```$ sudo mkdir /var/www/projectlamp``` to create a new directory in /var/www
- Assigned ownership of the directory to current user (me) using environment variable $USER ```$ sudo chown -R $USER:$USER /var/www/projectlamp```
- Created and opened new configuration file using ```$ sudo vi /etc/apache2/sites-available/projectlamp.conf```
```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost> 
```  
- Enabled the new Virtual Host ```$ sudo a2ensite projectlamp``` 
- Disabled defaule website ``` $ sudo a2dissite 000-default ```
- Tested config file to make sure it contains no errors ```sudo apache2ctl configtest```
- Reloaded Apache2 so that changes can take effect ```sudo systemctl reload apache2```
- Created and index.html file for projectlamp   
```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
![projectlamp index](https://user-images.githubusercontent.com/20668013/120053247-1b55e580-c021-11eb-831c-7baf27d5662f.JPG)

## Step 5 — Enable PHP on the website
- Edited the order of index files to give php precedence. ```sudo vim /etc/apache2/mods-enabled/dir.conf```
``` 
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
- Reloaded Apache so that changes could take effect ```sudo systemctl reload apache2```
- Created a new file to test that PHP is working fine ```vim /var/www/projectlamp/index.php```
```
<?php
phpinfo();
```
![php v](https://user-images.githubusercontent.com/20668013/120054336-46dbce80-c027-11eb-88aa-1f43d0edffc2.JPG)








 





