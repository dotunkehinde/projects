# Load Balancer Solution With Apache
## Introduction
In this project, we will enhance our Tooling website created in project 7 by adding a load balancer to distribute traffic between our Web Servers and allow users access to our website through only one domain.

### Task
Deploy and configure an Apache Load Balancer for the Tooling website solution on a seperate Ubuntu instance

### Prerequisites
The followinc instances have been created and configured in Project 7
1. Two RHEL8 Web Servers'
2. One MySQL DB Server (based on Ubuntu 20.04)
3. One RHEL8 NFS server  

![prereq](https://user-images.githubusercontent.com/20668013/122385515-d27cb700-cf64-11eb-88fc-062529bfc945.JPG)

## Configure Apache As A Load Balancer

1. Created an Ubuntu Server 20.04 EC2 Instance   

![aws list](https://user-images.githubusercontent.com/20668013/122386671-03a9b700-cf66-11eb-9203-526e65af29c1.JPG)

- Joined the Load balancer to an exisiting sercurity group that has port 80 alread opened.
- Installed Apache Load Balancer  on `Project-8-lb` server and configured it to point traffic to both Web Servers
```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```
Verified that Apache is up and running
```
sudo systemctl status apache2
```
![lb-up](https://user-images.githubusercontent.com/20668013/122387695-0953cc80-cf67-11eb-8b03-e5ce365bcaf3.JPG)
- Configured Load Balancing
```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2
```
- Verified that Load Balancer is working 
```
http://3.12.152.91/
```
![lb](https://user-images.githubusercontent.com/20668013/122388975-54221400-cf68-11eb-9ddb-2cd551f9c722.JPG)

## Optional Step - Configure Local DNS Names Resolution
```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
- Tested the Load Balancer and it still works!
![lb1](https://user-images.githubusercontent.com/20668013/122392413-c0524700-cf6b-11eb-8abb-830ecd05f729.JPG)



