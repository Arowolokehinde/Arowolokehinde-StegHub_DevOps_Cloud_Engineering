# MEAN Stack Documentation

## Introduction

The MEAN stack is a popular JavaScript stack used for building web applications. It stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Here's a brief overview of each component:

- **MongoDB**: MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It is a popular choice for web applications because of its scalability, flexibility, and performance.
- **Express.js**: Express.js is a web application framework for Node.js. It provides a set of features for building web applications and APIs, including routing, middleware support, and templating engines. Express.js simplifies the process of building web applications with Node.js by providing a high-level abstraction over the HTTP protocol.
- **AngularJS (or Angular)**: AngularJS is a JavaScript framework maintained by Google for building dynamic web applications. It extends HTML with additional attributes and provides a set of built-in directives for creating interactive user interfaces. AngularJS follows the Model-View-Controller (MVC) architecture, making it easy to organize code and build complex web applications.
- **Node.js**: Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It allows developers to run JavaScript code outside of a web browser, making it possible to build server-side applications with JavaScript. Node.js is known for its event-driven, non-blocking I/O model, which makes it well-suited for building scalable and high-performance web applications.

## Step 0: Prerequisites

1. **EC2 Instance**: Launch an EC2 instance of `t3.small` type with Ubuntu 22.04 LTS (HVM) in the `us-east-1` region using the AWS console.
![Screenshot 2024-10-04 152349](https://github.com/user-attachments/assets/18d93d96-0f71-4d43-8e29-a655adebf2fd)
![Screenshot 2024-10-04 152517](https://github.com/user-attachments/assets/9977c60e-6455-4489-82d8-4be7defe5826)

2. **SSH Key**: Attach an SSH key named `my-ec2-key` to access the instance on port 22.
3. **Security Group**: Configure the security group with the following inbound rules:
    - Allow traffic on port 80 (HTTP) from anywhere.
    - Allow traffic on port 443 (HTTPS) from anywhere.
    - Allow traffic on port 22 (SSH) from any IP address.
    - Allow traffic on port 3300 (Custom TCP) from anywhere.
![Screenshot 2024-10-04 152643](https://github.com/user-attachments/assets/56f7820d-70af-490d-96dc-744e40db0b44)

4. **SSH Connection**: Change the private SSH key permission and connect to the instance:
    ```sh
    chmod 400 my-ec2-key.pem
    ssh -i "my-ec2-key.pem" ubuntu@54.81.119.2
    ```
![Screenshot 2024-10-04 152936](https://github.com/user-attachments/assets/6ea03f06-7211-4f30-84aa-3d5e830e2e44)

## Step 1: Install Node.js
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.
1. **Update and Upgrade Ubuntu**:
    ```sh
    sudo apt update && sudo apt upgrade -y
    ```
![Screenshot 2024-10-04 153024](https://github.com/user-attachments/assets/31c4b8ce-1156-475c-a952-4c50287267e2)

2. **Add Certificates**:
    ```sh
    sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    ```
![Screenshot 2024-10-04 153119](https://github.com/user-attachments/assets/5db7fa74-28ca-4791-9795-6f6974f15592)

3. **Install Node.js**:
    ```sh
    sudo apt-get install -y nodejs
    ```
![Screenshot 2024-10-04 153249](https://github.com/user-attachments/assets/80272c09-e8b4-4788-8ea4-33528d1b4ed8)

## Step 2: Install MongoDB

In this application, book entries were stored in MongoDB, including the title, ISBN number, author, and the number of pages.
1. **Download MongoDB Public GPG Key**:
    ```sh
    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-archive-keyring.gpg
    ```
![Screenshot 2024-10-04 153410](https://github.com/user-attachments/assets/7fa89231-7757-407c-9b83-492d9aa19bdd)


2. **Add MongoDB Repository**:
    ```sh
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-archive-keyring.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
    ```
![Screenshot 2024-10-04 153410](https://github.com/user-attachments/assets/1fe71572-1823-45f6-a193-17ba5a92cd54)

3. **Update Package Database and Install MongoDB**:
    ```sh
    sudo apt-get update
    sudo apt-get install -y mongodb-org
    ```
![Screenshot 2024-10-04 153542](https://github.com/user-attachments/assets/cf1b31c7-c2be-4169-8fea-a8d29230e8ca)

4. **Start and Enable MongoDB**:
    ```sh
    sudo systemctl start mongod
    sudo systemctl enable mongod
    sudo systemctl status mongod
    ```
![Screenshot 2024-10-04 153635](https://github.com/user-attachments/assets/4ee68049-1daa-4766-9a2c-8d5c49686e74)

5. **Install `body-parser` Package**:
**body-parser** package is needed to help process JSON files passed in requests to the server.
    ```sh
    sudo npm install body-parser
    ```
![Screenshot 2024-10-04 153855](https://github.com/user-attachments/assets/6d197eca-05d4-43b4-9260-17d1de0a77eb)

![Screenshot 2024-10-04 154217](https://github.com/user-attachments/assets/859fa88e-6c56-417c-9cf7-2cb59609baaf)

6. **Create Project Root Folder named Books and initialize the root folder**:
    ```sh
    mkdir Books && cd Books
    npm init
    ```
![Screenshot 2024-10-04 154231](https://github.com/user-attachments/assets/691b71de-cdad-439f-8a9c-6356f29bfd07)

7. **Add `server.js` File to the Books folder and paste the code below**:
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
![Screenshot 2024-10-04 154333](https://github.com/user-attachments/assets/3e71c0fe-79bb-42c4-a71f-62eff197b96c)

## Step 3: Install Express and Set Up Routes

Express was utilized to transfer book data between our MongoDB database, while the Mongoose package offered a simple schema-based approach to model the application data. Mongoose was employed to define a schema for the database to manage the book records.

1. **Install Express and Mongoose**:
    ```sh
    sudo npm install express mongoose
    ```
![Screenshot 2024-10-04 154531](https://github.com/user-attachments/assets/ed219525-e9aa-4f89-a4ff-8dc13c0f3797)

2. **Create `apps` Folder and `routes.js` File in the Books folder**:
    ```sh
    mkdir apps && cd apps
    vim routes.js
    ```
    ![Screenshot 2024-10-04 154639](https://github.com/user-attachments/assets/a556fff0-afe9-49eb-93e6-02bd13f8e569)
copy and paste the code below into the routes.js
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
![Screenshot 2024-10-04 154613](https://github.com/user-attachments/assets/8ebafb98-f347-4ab3-96c1-4b1bc6842935)

3. **Create `models` Folder and `book.js` File in the apps Folder **:
    ```sh
    mkdir models && cd models
    vim book.js
    ```
![Screenshot 2024-10-04 154920](https://github.com/user-attachments/assets/fac23724-1125-4d94-a8fc-428971fde381)
Copy and paste the code below into book.js
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
![Screenshot 2024-10-04 154932](https://github.com/user-attachments/assets/104b0c0a-1c60-4298-b354-7fc40d72f296)

## Step 4: Access Routes with AngularJS
In this project, AngularJS was used to connect the web page with Express and perform actions on the book register.
1. **Change the directory back to ‘Books’ and create `public` Folder and `script.js` File**:
    ```sh
    cd ../..
    mkdir public && cd public
    vim script.js
    ```
![Screenshot 2024-10-04 155051](https://github.com/user-attachments/assets/97aa06a2-a76f-4917-a416-32fa2a140fc2)
Copy and paste the code below (controller configuration defined) into the script.js file.
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
![Screenshot 2024-10-04 155246](https://github.com/user-attachments/assets/73118079-efa9-4b99-a242-7b7df1a220b7)


2. ** In ‘public’ folder, create `index.html` File**:
    ```sh
    vim index.html
    ```
Copy and paste the code below into index.html file
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
![Screenshot 2024-10-04 155221](https://github.com/user-attachments/assets/704b7826-25f0-4ac1-92f3-e973c9635825)


3. **Change the directory back up to ‘Books’ and start the Server**:
    ```sh
    cd ..
    node server.js
    ```
![Screenshot 2024-10-04 155551](https://github.com/user-attachments/assets/5413ba7c-cac2-45b8-b9ee-2291995c92af)


![Screenshot 2024-10-04 155615](https://github.com/user-attachments/assets/32b9b610-6bb3-4faf-8a36-bf5ce7a2912f)

The server is now up and running, accessible via port 3300. You can test it using a browser with the public IP address or public DNS name.

The Book Register web application can now be accessed from the internet with a browser using the Public IP address or Public DNS name.
![Screenshot 2024-10-04 160158](https://github.com/user-attachments/assets/7e6c0891-f3a6-4eee-a028-ecddefe1cb75)

## Conclusion

The MEAN stack—comprising MongoDB, Express.js, AngularJS (or Angular), and Node.js—provides a powerful and cohesive set of technologies for building modern web applications.

Together, these technologies allow developers to use JavaScript throughout the entire development process, from front-end to back-end, promoting a unified and streamlined development workflow.
