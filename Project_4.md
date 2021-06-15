
 # MEAN Stack Deployment to Ubuntu in AWS

## Introduction
Mean stack is a combination of MongoDB, ExpressJS, Angular, and Node.js. 

### Task
Our task is to build and implement a simple Book Register web form using MEAN stack. 

## Step 0 - Preparing prerequisites
- Created a new EC2 instance with Ubuntu Server 20.04 LTS (HVM). 
- I connected to my new EC2 instance through AWS EC2 Instance Connect
![AWS - Instance Connect](https://user-images.githubusercontent.com/20668013/121419754-36e7b700-c964-11eb-8521-13af3f006c9e.JPG)

## Step 1 - Backend configuration
 - Updated my list of packages.
 ```
 $ sudo apt update  
 ``` 
 - Upgraded Ubuntu
 ```
 sudo apt upgrade
 ```
### Installation of Node.js on the server
- Installed Node.js with the command below
```
sudo apt-get install -y nodejs
```
 
## Step 2: Installation of MongoDB
- Ran the codes below
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
- Installed MongoDB with the commad below
```
sudo apt install -y mongodb
```
- Started the server with the command below
```
sudo service mongodb start
```
- Verified that the system is running by typing  
```
sudo systemctl status mongodb
```
![mongodb_running](https://user-images.githubusercontent.com/20668013/121422155-d6a64480-c966-11eb-990c-6f42cb3c46b8.JPG)
- Installed  `[npm](https://www.npmjs.com)` Node Package Manager
```
sudo apt install -y npm
```
- Installed 'body-parser' package
```
sudo npm install body-parser
```
- Created folder named 'Books'
```
mkdir Books && cd Books
```
- Initialized npm project in the books directory
```
npm init
```
- Added a file to it named server.js
```
vi server.js
```
- Copied and pasted the Web Server code below into the `server.js` file.
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
## Step 3: Install Express and set up routes to the server
- Installed Express using Mongoose to establish a schema for the database to store data of our book register. 
```
sudo npm install express mongoose
```
Created a folder named `apps` in 'Books' folder
```
mkdir apps && cd apps
```
- Created a file named `routes.js` in apps folder
```
vi routes.js
```
- Copied and pasted the code below into routes.js
```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```
- Created a folder named `models` in the 'apps' folder.
```
mkdir models && cd models
```
- Created a file named `book.js`
```
vi book.js
```
Copy and pasted the code below into 'book.js'
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```
## Step 4 - Access the routes with AngularJS

- Changed the directory back to 'Books'
```
cd ../..
```
- Created a folder named `public`
```
mkdir public && cd public
```
- Added a file named  `script.js`
```
vi script.js
```
- Copied and pasted the code below (controller configuration defined) into the script.js file. 
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
- Created a file named `index.html` in 'public' folder.
```
vi index.html
```
- Copied and pasted the code below into the index.html file.
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
- Changed directory back to 'Books'
```
cd ..
```
- Started the server byu running this command below
```
node server.js
```




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
- Edited EC2 Security Gorups and created an inbound rule to open TCP port 3300
- Typed ``` http://18.224.43.99:3300 ``` in browswer window 

![isbn](https://user-images.githubusercontent.com/20668013/121684792-1a0fc880-cab7-11eb-9d77-7c98833a85fe.JPG)
