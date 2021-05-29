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

