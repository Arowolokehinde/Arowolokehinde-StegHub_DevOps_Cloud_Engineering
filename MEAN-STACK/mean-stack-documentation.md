# MEAN Stack Documentation

## Introduction

The MEAN stack is a popular JavaScript stack used for building web applications. It stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Here's a brief overview of each component:

- **MongoDB**: MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It is a popular choice for web applications because of its scalability, flexibility, and performance.
- **Express.js**: Express.js is a web application framework for Node.js. It provides a set of features for building web applications and APIs, including routing, middleware support, and templating engines. Express.js simplifies the process of building web applications with Node.js by providing a high-level abstraction over the HTTP protocol.
- **AngularJS (or Angular)**: AngularJS is a JavaScript framework maintained by Google for building dynamic web applications. It extends HTML with additional attributes and provides a set of built-in directives for creating interactive user interfaces. AngularJS follows the Model-View-Controller (MVC) architecture, making it easy to organize code and build complex web applications.
- **Node.js**: Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It allows developers to run JavaScript code outside of a web browser, making it possible to build server-side applications with JavaScript. Node.js is known for its event-driven, non-blocking I/O model, which makes it well-suited for building scalable and high-performance web applications.

## Step 0: Prerequisites

1. **EC2 Instance**: Launch an EC2 instance of `t3.small` type with Ubuntu 22.04 LTS (HVM) in the `us-east-1` region using the AWS console.
2. **SSH Key**: Attach an SSH key named `my-ec2-key` to access the instance on port 22.
3. **Security Group**: Configure the security group with the following inbound rules:
    - Allow traffic on port 80 (HTTP) from anywhere.
    - Allow traffic on port 443 (HTTPS) from anywhere.
    - Allow traffic on port 22 (SSH) from any IP address.
    - Allow traffic on port 3300 (Custom TCP) from anywhere.
4. **SSH Connection**: Change the private SSH key permission and connect to the instance:
    ```sh
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@54.81.119.2
    ```

## Step 1: Install Node.js

1. **Update and Upgrade Ubuntu**:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
2. **Add Certificates**:
    ```sh
    sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    ```
3. **Install Node.js**:
    ```sh
    sudo apt-get install -y nodejs
    ```

## Step 2: Install MongoDB

1. **Download MongoDB Public GPG Key**:
    ```sh
    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
    ```
2. **Add MongoDB Repository**:
    ```sh
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```
3. **Update Package Database and Install MongoDB**:
    ```sh
    sudo apt-get update
    sudo apt-get install -y mongodb-org
    ```
4. **Start and Enable MongoDB**:
    ```sh
    sudo systemctl start mongod
    sudo systemctl enable mongod
    sudo systemctl status mongod
    ```
5. **Install `body-parser` Package**:
    ```sh
    sudo npm install body-parser
    ```
6. **Create Project Root Folder**:
    ```sh
    mkdir Books && cd Books
    npm init
    ```
7. **Add `server.js` File**:
    ```sh
    vim server.js
    ```
    ```javascript
    const express = require('express');
    const bodyParser = require('body-parser');
    const mongoose = require('mongoose');
    const path = require('path');
    const app = express();

    mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true, useUnifiedTopology: true })
      .then(() => console.log('MongoDB connected'))
      .catch(err => console.error('MongoDB connection error:', err));

    app.use(bodyParser.json());
    app.use(express.static(path.join(__dirname, 'public')));

    require('./apps/routes')(app);

    app.set('port', 3300);
    app.listen(app.get('port'), () => {
      console.log('Server up: http://localhost:' + app.get('port'));
    });
    ```

## Step 3: Install Express and Set Up Routes

1. **Install Express and Mongoose**:
    ```sh
    sudo npm install express mongoose
    ```
2. **Create `apps` Folder and `routes.js` File**:
    ```sh
    mkdir apps && cd apps
    vim routes.js
    ```
    ```javascript
    const Book = require('./models/book');
    const path = require('path');

    module.exports = function(app) {
      app.get('/book', async (req, res) => {
         try {
            const books = await Book.find({});
            res.json(books);
         } catch (err) {
            console.error(err);
            res.status(500).json({ error: 'Internal Server Error' });
         }
      });

      app.post('/book', async (req, res) => {
         try {
            const book = new Book({
              name: req.body.name,
              isbn: req.body.isbn,
              author: req.body.author,
              pages: req.body.pages
            });
            const result = await book.save();
            res.json({
              message: "Successfully added book",
              book: result
            });
         } catch (err) {
            console.error(err);
            res.status(500).json({ error: 'Internal Server Error' });
         }
      });

      app.put('/book/:isbn', async (req, res) => {
         try {
            const updatedBook = await Book.findOneAndUpdate(
              { isbn: req.params.isbn },
              req.body,
              { new: true }
            );
            if (!updatedBook) {
              return res.status(404).json({ error: 'Book not found' });
            }
            res.json({
              message: "Successfully updated the book",
              book: updatedBook
            });
         } catch (err) {
            console.error(err);
            res.status(500).json({ error: 'Internal Server Error' });
         }
      });

      app.delete('/book/:isbn', async (req, res) => {
         try {
            const result = await Book.findOneAndRemove({ isbn: req.params.isbn });
            if (!result) {
              return res.status(404).json({ error: 'Book not found' });
            }
            res.json({
              message: "Successfully deleted the book",
              book: result
            });
         } catch (err) {
            console.error(err);
            res.status(500).json({ error: 'Internal Server Error' });
         }
      });

      app.get('*', (req, res) => {
         res.sendFile(path.join(__dirname, '../public', 'index.html'));
      });
    };
    ```
3. **Create `models` Folder and `book.js` File**:
    ```sh
    mkdir models && cd models
    vim book.js
    ```
    ```javascript
    const mongoose = require('mongoose');

    const bookSchema = new mongoose.Schema({
      name: { type: String, required: true },
      isbn: { type: String, required: true, unique: true },
      author: { type: String, required: true },
      pages: { type: Number, required: true }
    });

    module.exports = mongoose.model('Book', bookSchema);
    ```

## Step 4: Access Routes with AngularJS

1. **Create `public` Folder and `script.js` File**:
    ```sh
    cd ../..
    mkdir public && cd public
    vim script.js
    ```
    ```javascript
    var app = angular.module('myApp', []);

    app.controller('myCtrl', function($scope, $http) {
      function getAllBooks() {
         $http({
            method: 'GET',
            url: '/book'
         }).then(function successCallback(response) {
            $scope.books = response.data;
         }, function errorCallback(response) {
            console.log('Error: ' + response.data);
         });
      }

      getAllBooks();

      $scope.add_book = function() {
         var body = {
            name: $scope.Name,
            isbn: $scope.Isbn,
            author: $scope.Author,
            pages: $scope.Pages
         };
         $http({
            method: 'POST',
            url: '/book',
            data: body
         }).then(function successCallback(response) {
            console.log(response.data);
            getAllBooks();
            $scope.Name = '';
            $scope.Isbn = '';
            $scope.Author = '';
            $scope.Pages = '';
         }, function errorCallback(response) {
            console.log('Error: ' + response.data);
         });
      };

      $scope.update_book = function(book) {
         var body = {
            name: book.name,
            isbn: book.isbn,
            author: book.author,
            pages: book.pages
         };
         $http({
            method: 'PUT',
            url: '/book/' + book.isbn,
            data: body
         }).then(function successCallback(response) {
            console.log(response.data);
            getAllBooks();
         }, function errorCallback(response) {
            console.log('Error: ' + response.data);
         });
      };

      $scope.delete_book = function(isbn) {
         $http({
            method: 'DELETE',
            url: '/book/' + isbn
         }).then(function successCallback(response) {
            console.log(response.data);
            getAllBooks();
         }, function errorCallback(response) {
            console.log('Error: ' + response.data);
         });
      };
    });
    ```
2. **Create `index.html` File**:
    ```sh
    vim index.html
    ```
    ```html
    <!DOCTYPE html>
    <html ng-app="myApp" ng-controller="myCtrl">
    <head>
      <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
      <script src="script.js"></script>
      <style>
         /* Add your custom CSS styles here */
      </style>
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
         <div ng-if="successMessage">{{ successMessage }}</div>
         <div ng-if="errorMessage">{{ errorMessage }}</div>
      </div>
      <hr>
      <div>
         <table>
            <tr>
              <th>Name</th>
              <th>Isbn</th>
              <th>Author</th>
              <th>Page</th>
              <th>Action</th>
            </tr>
            <tr ng-repeat="book in books">
              <td>{{ book.name }}</td>
              <td>{{ book.isbn }}</td>
              <td>{{ book.author }}</td>
              <td>{{ book.pages }}</td>
              <td><button ng-click="del_book(book)">Delete</button></td>
            </tr>
         </table>
      </div>
    </body>
    </html>
    ```

3. **Start the Server**:
    ```sh
    cd ..
    node server.js
    ```

The server is now up and running, accessible via port 3300. You can test it using a browser with the public IP address or public DNS name.

## Conclusion

The MEAN stack—comprising MongoDB, Express.js, AngularJS (or Angular), and Node.js—provides a powerful and cohesive set of technologies for building modern web applications. Together, these technologies allow developers to use JavaScript throughout the entire development process, from front-end to back-end, promoting a unified and streamlined development workflow.