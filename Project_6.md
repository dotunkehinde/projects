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

# Step 2 — Prepare the Database Server



