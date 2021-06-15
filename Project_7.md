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
1. Checked which port is used by NFS and opened it using Secureity Groups
```
rpcinfo -p | grep nfs
```
![rpcinfo](https://user-images.githubusercontent.com/20668013/122134988-95ff6d00-ce37-11eb-8144-76b9d0f1dd3b.JPG)

