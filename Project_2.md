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
- ``` http://18.117.187.158/ ```  
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
- http://18.117.187.158/info.php
```
![phpinfo](https://user-images.githubusercontent.com/20668013/120402058-2b2f3b80-c339-11eb-9f4e-5bc11d5de8c5.JPG)
- Removed php configuration file or security reasons.
```
 sudo rm /var/www/your_domain/info.php
```
## Step 6 — Retrieving data from MySQL database with PHP
- Created a database named example_database and user named example_user  
```
sudo mysql
```
```
mysql> CREATE DATABASE `example_database`;
```
```
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
- To give user 'example_user' permission over the database 'example_database'  
```
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%'; 
```
- Logging out of the database ``` mysql> exit ```
- Testing the user priviledge by logging in with creditials
``` 
mysql -u example_user -p 
```
- Displaying database earlier created
```
mysql> SHOW DATABASES;  
```
![show database](https://user-images.githubusercontent.com/20668013/120403796-dbeb0a00-c33c-11eb-8785-6728599fff69.JPG)

- Created a table named todo_list from the MySQL console
```
CREATE TABLE example_database.todo_list (
mysql>     item_id INT AUTO_INCREMENT,
mysql>     content VARCHAR(255),
mysql>     PRIMARY KEY(item_id)
mysql> );
```
- Inserting rows of content in the test table
```
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```
- Confirming that data was successfully saved to table
```
mysql>  SELECT * FROM example_database.todo_list;
```
![sql content](https://user-images.githubusercontent.com/20668013/120404392-2d47c900-c33e-11eb-8775-2b7f9fb46d4f.JPG)

- Exiting MySQL ``` mysql> exit ```  
- Creating a PHP Script that will query the database
```
nano /var/www/projectLEMP/todo_list.php
```
```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
![important item](https://user-images.githubusercontent.com/20668013/120405015-806e4b80-c33f-11eb-87d2-856057ad61d7.JPG)








 





