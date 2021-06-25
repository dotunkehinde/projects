# Devops Tooling Website Solution
### Task
Our task in this project is to implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible. 

Infrastructure: AWS
Webserver Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Programming Language: PHP
Code Repository: GitHub

## Step 1 - Prepare NFS Server
1. Spinned up a new EC2 instance with RHEL Linux 8 Operating System.
1. Configured LVM on the Server
- Formated the disks as `xfs` instead of ext4
- Created mount points on `/mnt` directory for the logical volumes as follows: Mount `lv-apps` on `/mnt/apps` - to be used by webservers, Mounted  `lv-logs` on `/mnt/logs` - to be used by webservers logs, Mounted `lv-opt` on `/mnt/opt` - to be used by Jenkins.
1. Installed NFS server, configure it to start on reboot and make sure it is up and running.
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
- Identified my subnet CIDR under Networking in EC2
- Set up permission that will allow the Web Servers to read, write and execute files on NFS
```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```
```
sudo systemctl restart nfs-server.service
```
- Configured access to NFS for clients within the same subnet 
```
sudo vi /etc/exports
```
```
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```
```
sudo exportfs -arv
```
<<<<<<< HEAD
1. Checked which port is used by NFS and opened ports TCP/UDP 2049, and TCP/UDP 111 in Security Groups
=======
1. Checked which port is used by NFS and opened it using Secureity Groups
>>>>>>> c9aa0ba6ec44437153ca7bf0eb7ac147d1da8147
```
rpcinfo -p | grep nfs
```
![rpcinfo](https://user-images.githubusercontent.com/20668013/122134988-95ff6d00-ce37-11eb-8144-76b9d0f1dd3b.JPG)

<<<<<<< HEAD
## Step 2 — Configure the database server
- Spinned up an Ubuntu instance on AWS EC2
- Installed MySQL server 
- Created a database named tooling
- Created a database user named webaccess
- Granted permisson to webaccess user on tooling to do anything from the webservers subnet cidr
```
CREATE DATABASE tooling;
CREATE USER `webaccess`@`172.31.0.0/20` IDENTIFIED BY 'dotman';
GRANT ALL ON tooling.* TO 'webaccess'@'172.31.0.0/20';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```  
![db1](https://user-images.githubusercontent.com/20668013/122318462-fd401e80-cf16-11eb-9b8e-e60629765ac6.JPG)
```
SELECT user FROM mysql.user;
```
![db2](https://user-images.githubusercontent.com/20668013/122318642-48f2c800-cf17-11eb-9d01-034aaa179143.JPG)
- Copied and pasted the folowing code into mysql
```
-- phpMyAdmin SQL Dump
-- version 5.0.2
-- https://www.phpmyadmin.net/
--
-- Host: 127.0.0.1
-- Generation Time: Oct 14, 2020 at 03:51 PM
-- Server version: 10.4.14-MariaDB
-- PHP Version: 7.4.9
SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */
;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */
;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */
;
/*!40101 SET NAMES utf8mb4 */
;
--
-- Database: `multi_login`
--
-- --------------------------------------------------------
--
-- Table structure for table `users`
--
CREATE TABLE `users` (
  `id` int(11) NOT NULL,
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `user_type` varchar(255) NOT NULL,
  `status` varchar(10) NOT NULL
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
--
-- Dumping data for table `users`
--
INSERT INTO `users` (
    `id`,
    `username`,
    `password`,
    `email`,
    `user_type`,
    `status`
  )
VALUES (
    1,
    'admin',
    '21232f297a57a5a743894a0e4a801fc3',
    'dare@dare.com',
    'admin',
    '1'
  );
--
-- Indexes for dumped tables
--
--
-- Indexes for table `users`
--
ALTER TABLE `users`
ADD PRIMARY KEY (`id`);
--
-- AUTO_INCREMENT for dumped tables
--
--
-- AUTO_INCREMENT for table `users`
--
ALTER TABLE `users`
MODIFY `id` int(11) NOT NULL AUTO_INCREMENT,
  AUTO_INCREMENT = 3;
COMMIT;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */
;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */
;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */
;
```
```
use tooling
SELECT * FROM users;
```
![db3](https://user-images.githubusercontent.com/20668013/122318782-8d7e6380-cf17-11eb-8c18-eb95127ccc6f.JPG)

## Step 3 — Prepare the Web Servers
- Launched a new EC2  instance with RHEL 8 OS
- Installed the NFS client
```
sudo yum install nfs-utils nfs4-acl-tools -y
```
- Mounted /var/www and targeted teh NFS server's export for apps
```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
- Verified that NFS was mounted successfully by runnign `df -h` 
![nfs](https://user-images.githubusercontent.com/20668013/122319617-bc490980-cf18-11eb-8746-65bab9bf7cec.JPG)
- Edited the `/etc/fstab` file and added the following line
```
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```
![ws](https://user-images.githubusercontent.com/20668013/122320004-658fff80-cf19-11eb-8131-5aa75b1e10f9.JPG)
- Installed Apache
```
sudo yum install httpd -y
```
- Installed PHP
- Repeated the steps above for 2 additional Web servers
- Mounted the log folder for Apache to the NFS server
- Downloaded the tooling source code from Darey.io Github account  and uploaded unto the `/var/www/html` directory on the web server using WinSCP
- Opened port 80 on the webserver
- Disabled SELINUX
- 
```
sudo setenforce 0
```
- Edited the config file `sudo vi /etc/sysconfig/selinux` and set  `SELINUX=disabled`.
- Updated the website's configuration to connect the databse in  `functions.php` file and applied the `tooling-db.sql` script
- Created new admin in MySQL database with username `myuser` and password `password`
- Inserted the code below into the database
```
INSERT INTO `users` (`id`,`username`,`password`,`email`,`user_type`,`status`) VALUES ('3','myuser','5f4dcc3b5aa765d61d8327deb882cf99','user@mail.com','admin',1);
```
- Opened the website in browser
```
http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php
```
![propitix1](https://user-images.githubusercontent.com/20668013/122320849-d1bf3300-cf1a-11eb-91ec-39de44ce2ab8.JPG)
- Username: myuser
- Password: password  

<<<<<<< HEAD

=======
>>>>>>> c9aa0ba6ec44437153ca7bf0eb7ac147d1da8147
=======
![propitix2](https://user-images.githubusercontent.com/20668013/122381783-343b2200-cf61-11eb-9186-f96b25905f29.JPG)

- Tested on the other 2 webservers

![image](https://user-images.githubusercontent.com/20668013/122382170-9136d800-cf61-11eb-84f8-a15108eccc72.png)

![propitix4](https://user-images.githubusercontent.com/20668013/122382531-ea9f0700-cf61-11eb-923c-d556941d37f3.JPG)


## Challenges Encountered

- I had problems connecting the with my DB server. I had to edit the functions.php file from the website to point to the private IP address of my DB Server.

```
<?php 
session_start();

// connect to database
$db = mysqli_connect('172.31.39.229', 'webaccess', 'dotman', 'tooling');

// Check connection
// if (mysqli_connect_errno()) {
// echo "Failed to connect to MySQL: " . mysqli_connect_error();
// exit();
// }
// else{
// echo "connected";
// }

// variable declaration
$username = "";
```



>>>>>>> 33c87a06bd8b9a08596db148deec3d395558d516
