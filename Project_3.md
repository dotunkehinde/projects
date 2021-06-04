# Simple To-Do application on MERN Web Stack

## Introduction
This project implements a web solution based on MERN stack in AWS Cloud. A MERN Web stack comprises of MongoDB, ExpressJS, ReactJS, and Node.js. 

### Task
Our task is to build a simple ToDo application that creates ToDo lists. 

## Step 0 - Preparing prerequisites
- Created a new EC2 instance with Ubuntu Server 20.04 LTS (HVM). 
- Downloaded MobaXterm, a multitool terminal for Windows. 
- I connected to my new EC2 instance through MobaXterm
![moboxterm](https://user-images.githubusercontent.com/20668013/120836773-d8829900-c55d-11eb-87d5-e1e41b9cd7ed.JPG)

## Step 1 - Backend configuration
 - Updated my list of packages.
 ```
 $ sudo apt update  
 ``` 
 - Upgraded Ubuntu
 ```
 sudo apt upgrade
 ```
 - Got the location of the Node.js software from the Ubuntu repositories

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
### Install Node.js on the server
- Installed Node.js with the command below
```
sudo apt-get install -y nodejs
```
This installs both Node.js and NPM, a package manager for installing Node packages and manage dependency conflicts.
- Verified the installation with the command below  
``` 
node -v 
```
```
npm -v 
```
![verify node](https://user-images.githubusercontent.com/20668013/120838187-70cd4d80-c55f-11eb-9f1d-b3fd7e03a556.JPG)

### Application Code Setup
- Created a new directory for our To-Do project
```
mkdir Todo
```
- Changed directory to the newly created ToDo directory
```
cd Todo
```
- Initialized the project with the command below
```
npm init
```
![initialize nodejs](https://user-images.githubusercontent.com/20668013/120839028-5f387580-c560-11eb-8754-3d5dd4a316bf.JPG)

### Install ExpressJS
- Installed express using npm
```
npm install express
```
- Created index.js with the command below
```
touch index.js
```
- Installed the 'dotdev' module with the code below
```
npm install dotenv
```
- Opened the index.js file with the command below
```
vim index.js
```
- Typed the code below into index.js
```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
- Started server with the command below
```
node index.js
```
![server-running](https://user-images.githubusercontent.com/20668013/120840191-cf93c680-c561-11eb-9a97-773dd05be336.JPG)
- Edited EC2 Security Gorups and created an inbound rule to open TCP port 5000
- Typed ``` http://18.224.43.99:5000 ``` in browswer window
![Welcome Express](https://user-images.githubusercontent.com/20668013/120840747-8728d880-c562-11eb-95e4-36206c6c57b3.JPG)

### Routes
Our To-Do application needs to be able to 
1. Create a new task
2. Display a list of all tasks
3. Delete a completed task

Each task will be associated with some particular endpoint and will use different standard HTTP request methods; POST, GET, DELETE

For each task, we need to create routes that will define various endpoints that the To-Do app will depend on. We need to create a folder for the routes

- Created routes folder with the command below
```
mkdir routes
```
- Changed to the routes folder
``` 
cd routes
```
- Created a file api.js with the command below
```
touch api.js
```
- Opened created file with the command below
```
vim api.js
```
- Copied the code below into the file
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```
### Models
To create a Schema and a model, I installed mongoose a Node.js package that makes it easier to work with mongodb.

- Changed directory back to the Todo folder and install Mongoose
```
cd ..
```
```
npm install mongoose
```
- Created a new folder called models and changed directory to newly created one.
```
mkdir models
```
```
cd models
```
- Created a file called todo.js inside the models folder
``` 
touch todo.js
```
#### Note
All three commands above can be defined in one line as shown below
```
mkdir models && cd models && touch todo.js
```
- Opened the created file using ``` vim todo.js``` and pasted the code below into it.
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
We now need to update our routes from the file api.js in 'routes' directory to make use of the new model.
- Opened api.js in the Routes directory with ``` vim api.js```
- Deleted the code inside with ```:%d``` and replaced it with the code below
```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```
### MongoDB Database








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

#### To access this project, please click [here!](ec2-18-191-149-182.us-east-2.compute.amazonaws.com)








 





