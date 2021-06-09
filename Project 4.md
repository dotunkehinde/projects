
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
- Edited EC2 Security Gorups and created an inbound rule to open TCP port 5000
- Typed ``` http://18.224.43.99:5000 ``` in browswer window 








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

- Created an account on Mongodb.com  
![mongodb](https://user-images.githubusercontent.com/20668013/120846533-29988a00-c56a-11eb-9ddd-c98f7f786e76.JPG)
- Created a file in Todo directory with the name ```.env```
```
touch .env
vi .env
```
- Updated index.js file to reflect the use of ```.env``` so that Node.js can connect to the database.
- Deleted the existing content of the file and updated it with the code below

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
- Started server using ``` node index.js```  
![tododb connect](https://user-images.githubusercontent.com/20668013/120850167-f99fb580-c56e-11eb-8ae9-5e64add0d993.JPG)

### Testing Backend Code without Frontend using RESTful API
- Downloaded and installed Postman to test our API.
- Opened Postman and created a POST request to the API
```
http://<PublicIP-or-PublicDNS>:5000/api/todos
```
- I set header key ```Content-Type``` as ```application/json```
- Created a POST request ``` http://18.224.43.99:5000/api/todos ```
![postman](https://user-images.githubusercontent.com/20668013/120853002-f1497980-c572-11eb-94ca-9af91db613fc.JPG)
- Created a GET request to my API on ``` http://18.224.43.99:5000/api/todos ```  
This request will retrieve all existing records from the database of our To-do application 
![postman2](https://user-images.githubusercontent.com/20668013/120853597-d6c3d000-c573-11eb-8029-fd63c4284eae.JPG)
- Created a DELETE request to my API on ``` http://18.224.43.99:5000/api/todos/<id> ```
## Step 2 - Frontend Creation
In this step, we are going to create a user interface for a Web client to interact with the application via API.
- Changed directory to the Todo folder ```cd Todo ``` and ran the code below 
```
npx create-react-app client
```
### Running a React App
Installed some dependencies that need to be installed. 
1. Concurrently is used to run more than one command simultaneosly for the same terminal window.
```
npm install concurrently --save-dev
```
2. Intalled nodemon. Nodemon is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
```
npm install nodemon --save-dev
```
3. In Todo folder, opened the ```package.json``` file. Changed the highlighted part of the screenshot below and replaced it with the code that appears below it.   
![packagejson](https://user-images.githubusercontent.com/20668013/120867697-5ceb1100-c58a-11eb-9449-cc6fb7cfd9c6.JPG)
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

### Configure Proxy in ```package.json```
1. Changed directory to 'client'
```
cd client
```
2. Opened the ```package.json``` file

```
vi package.json
```
3. Added the key value pair in the package.json file ``` "proxy": "http://localhost:5000" ```
This enables us to be able to access the application directly from the browser by simly calling the server url like ``` http://localhost:5000``` rather than always including the entire path like ```http://localhost:5000/api/todos

4. Ran the code below ensuring I was inside the Todo folder.
```
npm run dev
```
![complied succesfully](https://user-images.githubusercontent.com/20668013/120868899-fca99e80-c58c-11eb-89ec-5d6a9e29ca3e.JPG)

5. Added new inboud rule in EC2 security group to allow inbound TCP connections on port 3000
6. Entered ``` http://18.224.43.99:3000/ ``` on browser window  

![reactapp](https://user-images.githubusercontent.com/20668013/120868992-3c708600-c58d-11eb-97ea-7d296f28d180.JPG)

### Creating your React Components
- Changed directory to client
```
cd client
```
- Moved to the src directory
```
cd src
```
- Created another folder called components in the src folder
```
mkdir components
```
- Moved into the components directory with 
```
cd components
```
- Created three files inside the components directory namely; ```Input.js```, ```ListTodo.js```, and ```Todo.js```
```
touch Input.js ListTodo.js Todo.js
```
- Opened ```Input.js```
```
vi Input.js
```
- Pasted the following code into Input.js
```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```
- Installed axios in the ```clients``` folder
```
npm install axios
```
- Pasted the following code into ```ListTodo.js``` in the ```Todo/clients/src/components``` folder
```
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
```
- Pasted the code below into the ```Todo.js``` folder
```
import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```
- Copy and pasted the code below into the ```App.js``` file in the ```Todo/client/src``` folder
```
import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```
- Pasted the code below into the ```App.css``` file in the ```Todo/client/src/``` folder
```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
- Pasted the code below into the ```index.css``` file in the ```Todo/client/src``` directory
```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
- Ran ``` npm run dev ``` from the ```Todo``` directory

![todoapp](https://user-images.githubusercontent.com/20668013/120870913-3204bb00-c592-11eb-9fe3-6326fde10ffa.JPG)

 

 





