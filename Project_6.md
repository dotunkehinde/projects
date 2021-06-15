# Web Solution With WordPress
- Our task in this project is to  prepare storage infrastructure for two Linux Servers and implement a basic web solution using WordPress. 
# Step 1 - Prepare a Web Server
- Launched a RedHat Linux EC2 instance that will serve as "Web Server"
- Created 3 10 GiB volumes in the same AZ as the Web Server
- Attached all three volumes one by one to the Web Server EC2 instance
- Opened the Linux terminal to begin configuration of Web Server
- Used `lsblk` command to inspect what block devices are attached to teh server.   
![ws lsblk](https://user-images.githubusercontent.com/20668013/121974097-9552d200-cd76-11eb-83f5-1455568795b6.JPG)
 
- Used `df -h` command to see all mounts and free space on the server
- Used `gdisk` utility to create a single partition on each of the 3 disks. 
```
sudo gdisk /dev/xvdf
```
- Used `lsblk` utility to view the newly configured partition on each of the 3 disks  
![g-partition](https://user-images.githubusercontent.com/20668013/121975623-f5974300-cd79-11eb-8130-24d0e7ad66b7.JPG)  
- Installed `lvm2` package using `sudo yum install lvm2`
- Ran sudo `lvmdiskscan` to check for available partitions.
- Used pvcreate uitilty to mark each of the 3 disks as a physical volumn to be used by LVM
```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
- Verified that Physical volume has been created successfully by running `sudo pvs`  
![pvs](https://user-images.githubusercontent.com/20668013/121976044-e82e8880-cd7a-11eb-9fd1-2bf02c178437.JPG)
- Used `vgcreate` utility to add all 3 PVs to a volume group (VG) named webdata-vg
```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
- Verified that VG ahs been created successfully by running `sudo vgs`  
![vgs](https://user-images.githubusercontent.com/20668013/121976712-4740cd00-cd7c-11eb-930d-89b65969051e.JPG)
- Used `lvcreate` utility to create 2 logical volumes; apps-lv and logs-lv. apps-lv will be used to store data for the website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
- Verified that logical volume had been created successfully by running `sudo lvs`  
![lvs](https://user-images.githubusercontent.com/20668013/121977091-0f865500-cd7d-11eb-8aac-52a115af7ea9.JPG)
- Verified the entire setup 
```
sudo vgdisplay -v
sudo lsblk
```
*Output*
```
[ec2-user@ip-172-31-4-18 dev]$ sudo vgdisplay -v
  --- Volume group ---
  VG Name               webdata-vg
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <29.99 GiB
  PE Size               4.00 MiB
  Total PE              7677
  Alloc PE / Size       7168 / 28.00 GiB
  Free  PE / Size       509 / <1.99 GiB
  VG UUID               cfzktk-ibYx-T0xU-qUUI-3NEf-YDcz-AqjUUp

  --- Logical volume ---
  LV Path                /dev/webdata-vg/apps-lv
  LV Name                apps-lv
  VG Name                webdata-vg
  LV UUID                rk6ngk-9ifZ-74iW-wogr-VAUB-nud6-W0qQoc
  LV Write Access        read/write
  LV Creation host, time ip-172-31-4-18.us-east-2.compute.internal, 2021-06-15 00:55:43 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/webdata-vg/logs-lv
  LV Name                logs-lv
  VG Name                webdata-vg
  LV UUID                j1ge25-cLcf-WOoY-IAKl-jiRo-W7mj-4yfRMc
  LV Write Access        read/write
  LV Creation host, time ip-172-31-4-18.us-east-2.compute.internal, 2021-06-15 00:55:56 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Physical volumes ---
  PV Name               /dev/xvdh1
  PV UUID               xeHrls-K5Cg-WiUs-zJkS-mkm0-Qbdk-uOFN53
  PV Status             allocatable
  Total PE / Free PE    2559 / 0

  PV Name               /dev/xvdg1
  PV UUID               IU2Kbk-tIdW-qZwU-JUb0-hRE1-UKcG-ppUfL9
  PV Status             allocatable
  Total PE / Free PE    2559 / 509

  PV Name               /dev/xvdf1
  PV UUID               eogcTG-429t-a7us-C9dU-HFpi-nTzx-7VjYK2
  PV Status             allocatable
  Total PE / Free PE    2559 / 0

[ec2-user@ip-172-31-4-18 dev]$
```
![lsblk](https://user-images.githubusercontent.com/20668013/121977475-077ae500-cd7e-11eb-84a6-522f1329d373.JPG)
- Used `mkfs.ext4` to format the logical volumes with ext4 filesystem
```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
- Created /var/www/html directory to store website files
```
sudo mkdir -p /var/www/html
```
- Created /home/recovery/logs to store backup of log data
```
sudo mkdir -p /home/recovery/logs
```
- Mounted /var/www/html on apps-lv logical volume
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
- Used `rsync` utility to backup all the files in the log directory /var/log into /home/recovery/logs. *This should be done before mounting the file system*
```
sudo rsync -av /var/log/. /home/recovery/logs/
```
- Mounted /var/log on logs-lv logical volume. *This deletes all existing data on /var/log hence the need for the step above*'
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
- Restored log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/. /var/log
```
- Updated `/etc/fstab` file so that the mount configuration will persist after a restart of the server. The UUID of the device is used to upadate the `/etc/fstab` file.
```
sudo blkid
```
- Edited `/etc/fstab`. Copied the UUID into fstab
```
vi /etc/fstab
```
- Tested the configuration and reloaded the daemon
```
sudo mount -a
sudo systemctl daemon-reload
```
- Verified that the setup is running using `df -h`  
![df](https://user-images.githubusercontent.com/20668013/121979851-e072e200-cd82-11eb-8c4c-658cdb344c17.JPG)

## Step 2 — Prepare the Database Server
Launched a second RedHat EC2 instance that will have a role as a 'DB Server' and repeated the same steps as for the Web Server, but instead of `apps-lv` I created `db-lv` and monted it to `/db` directory instead of `/var/www/html/`  
![db](https://user-images.githubusercontent.com/20668013/121983143-bf14f480-cd88-11eb-8078-9819172f833f.JPG)
## Step 3 — Install Wordpress on your Web Server EC2
- Updated repository on the Web Server
```
sudo yum -y update
```
- Installed wget, Apache and it's dependencies
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
- Started Apache
```
sudo systemctl enable httpd
sudo systemctl start httpd
```
- Installed PHP and it's dependencies
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```
- Restarted Apache
```
sudo systemctl restart httpd
```
- Downloaded Wordpress and Copied into `var/www/html`
```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
cp -R wordpress /var/www/html/
```
- Configured SELinux Policies
```
 sudo chown -R apache:apache /var/www/html/wordpress
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 sudo setsebool -P httpd_can_network_connect=1
```
## Step 4 — Install MySQL on your DB Server EC2
```
sudo yum update
sudo yum install mysql-server
```
- Verified that the service is up and running 
```
sudo systemctl status mysqld
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
![mysqld](https://user-images.githubusercontent.com/20668013/121985471-b2929b00-cd8c-11eb-8eab-cf9ece3e3f71.JPG)
## Step 5 — Configure DB to work with WordPress
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
![wp-db](https://user-images.githubusercontent.com/20668013/121985958-9e9b6900-cd8d-11eb-804e-15df2fc86a1b.JPG)
## Step 6 — Configure WordPress to connect to remote database.
- Opened MySQL port 3306 on DB Server EC2. I allowed access to the DB server only from the Web Server's IP address.
- Installed MySQL client and tested that connection can be made from WebServer to DB Server using  `mysql-client`
```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
1. Verified that I can sucessfully execute `SHOW DATABASES` command to see a list of existing databases.
![sh db](https://user-images.githubusercontent.com/20668013/121987100-b247cf00-cd8f-11eb-9413-e19288ffe21d.JPG)
2. Changed permissions and configuraton so that Apache could use WordPress
3. Enabled TCP port 80 in Inbound rules configuration for Web Server.
4. Accesed the Webserver through a browser
```
http://<Web-Server-Public-IP-Address>/wordpress/
```
![wp-install](https://user-images.githubusercontent.com/20668013/121989965-e7a2eb80-cd94-11eb-8ed2-eb58ba241c31.JPG)  
![wp-install 3](https://user-images.githubusercontent.com/20668013/121990910-824ffa00-cd96-11eb-93ac-15833345d3f2.JPG)  

![wp-install 4](https://user-images.githubusercontent.com/20668013/121991096-e2df3700-cd96-11eb-8183-d29984667d15.JPG)
