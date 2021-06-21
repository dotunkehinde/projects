# Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

### Task
Our task in this project is to enhance Project 8 by adding a Jenkins Server to automatically deploy source code changes from Git to NFS Server.  
Our architecture would look like the diagram below    
![jenk](https://user-images.githubusercontent.com/20668013/122690688-f3723000-d222-11eb-9b5d-71c63d5f9a3f.JPG)

## Step 1 - Install Jenkins server
1. Createe an AWS EC2 server based on Ubuntu Server 20.04 LTS and named it "Jenkins"
2. Installed JDK because Jenikins is a Java-based application
```
sudo apt update
sudo apt install default-jdk-headless
```
1. Installed Jenkins 
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
1. Verified that Jenkins is up and running
```
sudo systemctl status jenkins
```
![jenkins verify](https://user-images.githubusercontent.com/20668013/122691062-21587400-d225-11eb-8108-443b122b7ef0.JPG)
1. Created an inbound rule in the EC2 Security group to allow traffic through port 8080
2. Performed initial Jenkins setup
```
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
```
![unlock jenkins](https://user-images.githubusercontent.com/20668013/122691157-d25f0e80-d225-11eb-9c35-0cd80705a5d6.JPG)

![jenkins ready](https://user-images.githubusercontent.com/20668013/122691497-edcb1900-d227-11eb-8dbe-99798995e36c.JPG)

## Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks
1. Enabled webhooks in GitHub repository settings
2. Created "Freestyle Project" in Jenkins web console 
3. Connected to GitHub repository by copying the url from the repository
4. Provided the linke to Tooling GitHub repository and credentials so that Jenkins could access files in the repository
5. Saved the configuration and then clicked "Build Now"
6. Build was successful  

![build1](https://user-images.githubusercontent.com/20668013/122693489-fc1e3280-d231-11eb-87aa-6f260febe6fe.JPG)

1. Checked the console output  
![build](https://user-images.githubusercontent.com/20668013/122693519-19eb9780-d232-11eb-8960-0a22bd5ca96f.JPG)
1. Configured the project to trigger the job from GitHub webhook by check-marking "GitHub hook trigger for GITScm polling"
2. Configured "Post-build Actions" to archive all teh files - files that resulted from the build also called "artifacts" by typing 2 astericks under "Files to archieve" in " Post-Build Actions"
3. Edited and commited the README.md file
4. Build was automatically started on Jenkins    
![build3](https://user-images.githubusercontent.com/20668013/122694295-d181a900-d234-11eb-9df2-f383f67ee11a.JPG)
![build4](https://user-images.githubusercontent.com/20668013/122694308-d9d9e400-d234-11eb-84e5-4bc99a6c73b3.JPG)

## Step 3 - Configure Jenkins to copy files to NFS server via SSH
1. Installed "Publish Over SSH" plugin on Jenkins
2. Selected "Manage Jenkins" on the main dashboard and clicked "Manage Plugins". Typed "Publish Over SSH" on the "Available" tab search and installed the plugin.
3. Configured the project to copy artifacts over to NFS server.
4. Selected "Manage Jenkins" and chose "Configure System" menu item on the main dashboard
5. Scrolled down to "Publish over SSH" plugin configuration section and configured it wo be able to connect to NFS server
-  Provided a private key that is used to conenct to the NFS server through SSH
-  Entered the private IP of NFS server as Hostname
-  Entered `ec2-user` as Username
-  Entered `/mnt/apps` as the mounting point to recieve files on the NFS server
-  Tested configuration by clicking "Test Configuration"
![nfs-connect](https://user-images.githubusercontent.com/20668013/122695740-36d79900-d239-11eb-99b7-8d407f00c38a.JPG)
1. Saved the configuration
2. Opened Jenkins project configuration page and added one "Post-build Action"
- `Send build artifacts over SSH`
1. Saved the configuration
2. Edited and commited the Readme file on GitHub *Added `Welcome` to the first line*
3. Checked the NFS Server to see if changes were reflected  
![welcome](https://user-images.githubusercontent.com/20668013/122703138-ce90b380-d248-11eb-9081-48e8f158bc19.JPG)
![welcome2](https://user-images.githubusercontent.com/20668013/122703149-d6e8ee80-d248-11eb-9817-ba2e46b9e892.JPG)
![welcome3](https://user-images.githubusercontent.com/20668013/122703155-da7c7580-d248-11eb-9aac-865442d6fc85.JPG)



