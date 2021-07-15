# **Build a Book Register Form using the MEAN Web Stack**

This Project covers the implementation of developing a To-Do app using the MEAN Stack from start to finish and deploying to **AWS** (Amazon Web Services). The MEAN Stack is a Technology Stack. A Technology stack comprises of framework/tools that are used in building a software application. The MEAN Acronym stands for MongoDB, ExpressJS, Angular, Node.

- MongoDB - MongoDB is a document-based NoSQL database that stores data in form of documents. It allows for storage of data in semi-structured and unstuctured formats.

- ExpressJ - ExpessJS is a server side web framework that is built for NodeJS. It comes with a lot of features out of the box and it enables the development process to be more faster and efficient.

- Angular- Angular is a Front-end Framework built with Javascript for building user interfaces or client-facing apps.

- NodeJS - NodeJS is a javascript runtime that is built on the v8 engine, it enables you to run javascript on your local machine rather than on the browser.

## **Create an EC2 instance on your AWS Dashboard**

You have to create a Linux virtual server on AWS. There are several features and services that AWS offer but for this project, I'll be using the ec2 instance. If you're using windows, you have to download a CLI (Command Line Interface) such as Git, Putty, Termius, MobaXterm that is Linux-enabled so you can run basic linux commands. I watched this [video](https://www.youtube.com/watch?v=xxKuB9kJoYM&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=7) to create a Linux virtual server, also i need to add that the Linux distro we're using is Ubuntu.

When we're done creating the server, we first have to add permissions to our private key using this command.

![1](https://user-images.githubusercontent.com/47898882/125817420-2a9b775b-eea9-4ef8-822d-84283e0e30b6.JPG)

If that works out successfully, you can then connect to the server using the ssh command.

![2](https://user-images.githubusercontent.com/47898882/125817444-b289df98-98e1-4089-989c-37bea1a79429.JPG)

Now that our Virtual Server is up and running we can start building our application.

# **Install NodeJS & other Dependencies**

- Update and Upgrade the _apt_ package manager in order to have the newest updated version

```
$ sudo apt update
$ sudo apt upgrade
```

- Install NodeJS using the _apt_ command

```
$ sudo apt install nodejs -y
```

- Install npm and body-parser using the _apt_ command. npm is the pacage manager for NodeJS while body-parser allows us to process JSON files passed as a request to the server.

```
$ sudo apt install npm body-parser
```

# **Install MongoDB**

- Add the key for MongoDB using the command below

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```

- To check if MongoDB has been added to the list of keys, use `apt-key list` command

![1](https://user-images.githubusercontent.com/47898882/125859133-1e766ad8-4023-48a5-8b9a-ce7c9f3fca41.JPG)

- Download the MongoDB Repository in Ubuntu. It'll be stored in the `etc/apt/sources.list.d` directory.

```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

- Install MongoDB using the _apt_ command

```
$ sudo apt install mongodb -y
```

- Check status of MongoDB to see if it is active & enabled

![2](https://user-images.githubusercontent.com/47898882/125859645-1de930d9-444e-446e-8972-f0893ea3f245.JPG)

# **Application Code Setup**

- Create a folder called books using the _cd_ command and in that folder initilize your project using `npm init`

```
$ mkdir Books && cd Books
$ npm init
```

- Create a file called server.js and use the `vi` command to enter into the file and write into the file.

```
$ touch server.js && vi server.js
```

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

# **Install ExpressJS & Set up routes to the Server**

- Install Express using the _npm_ command.

```
$ npm install express
```

- Create a folder called apps using the _cd_ command, and in that folder create a file called routes.js

```
$ mkdir apps && cd apps && touch routes.js
```

- Use the `vi` command to enter into the file and write into the file.

```
$ vi routes.js
```

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

- In the same _apps_ directory, create a folder called models

```
$ mkdir models && cd models
```

- Create a file called book.js in the models folder and write into the file using the `vi` command

```
$ touch book.js && vi book.js
```

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

# **Access the Routes using Angular**

- Go back to the Books directory and create a folder called _public_

```
$ mkdir public
```

- Create a file called script.js in the public folder and write into the file using the `vi` command

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

- Create a file called index.html in the public folder and write into the file using the `vi` command

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

- Change directory to Books and run _node server.js_

**Note** - Do not Forget to set inbound rules to allow connection from port 3300, as that is the port the server will be listening from

![3](https://user-images.githubusercontent.com/47898882/125862449-3066dcdb-860c-4b92-8fe9-cd46da5ef996.JPG)

- Test on your browser using the public IP address or DNS name. `http://<Public-IP-Address>:3300 or http://DNS-name:3300`.

![4](https://user-images.githubusercontent.com/47898882/125862629-5443c234-d707-4d1d-a4b1-54fc3003c496.JPG)

- Add more books to the form at your own discretion.

**This is how to build a book register using the MEAN Stack, and deploy to AWS EC2.**.
