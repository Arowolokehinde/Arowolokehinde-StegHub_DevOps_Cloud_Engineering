### MERN Web Stack Implementation in AWS

### Introduction

### The MERN stack is widely used for building dynamic web applications, utilizing a combination of JavaScript technologies:

- **MongoDB**: A NoSQL document-oriented database offering flexible data storage.
- **Express.js**: A minimal web framework for Node.js, designed to simplify API and web application development.
- **React.js**: A powerful library for creating interactive user interfaces.
- **Node.js**: A JavaScript runtime that enables server-side JavaScript execution.

This guide provides a detailed walkthrough of setting up and using each component of the MERN stack to develop robust web applications.


## Step 0: Prerequisites

__1.__ EC2 Instance of t3.small type and Ubuntu 24.04 LTS (HVM) was lunched in the us-east-1 region using the AWS console.
The choice of the instance type was based on the following:

- **Memory:** The t3.small instance offers more memory than the t2.micro, which is advantageous for applications that require more memory to operate efficiently.

- **Burst Capability:** While both instances offer burstable CPU performance, the t3 instances have a more flexible burst model, allowing for more sustained performance during burst periods. This is important for workloads that require consistent performance over longer periods.

- **Performance:** While both instances offer burstable performance, the t3.small typically provides better baseline performance compared to the t2.micro. This might be necessary for applications that require a bit more processing power.
![Screenshot 2024-10-03 113526](https://github.com/user-attachments/assets/0f075b09-80eb-433c-a667-f0929d13200e)


**2** Attached SSH key named __my-ec2-key__ to access the instance on port 22

-  The security group was configured with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet.
- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.
- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.
- Allow traffic on port 5000 (Custom TCP) with source from anywhere.
- Allow traffic on port 3000 (Custom TCP) with sourec from anywhere.

![Screenshot 2024-10-03 114116](https://github.com/user-attachments/assets/4a64659e-999d-4d44-970f-fc66a3517c59)


**4** The private ssh key permission was changed for the private key file and then used to connect to the instance by running
```bash
chmod 400 my-ec2-key.pem
```
```bash
ssh -i "my-ec2-key.pem" ubuntu@ip
```
Where __username=ubuntu__ and __public ip address=ip
![Screenshot 2024-10-03 114501](https://github.com/user-attachments/assets/6b3cc363-ed78-4eb2-89c7-7309f3da0936)


## Step 1 - Backend Configuration

**1.** Update and upgrade the server’s package index 

```bash
sudo apt update && sudo apt upgrade -y
```
![Screenshot 2024-10-03 114426](https://github.com/user-attachments/assets/525a6251-33ce-4b05-9811-4ee15294b02d)

**2.** Get the location of Node.js software from ubuntu repositories.

```bash
curl fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
![Screenshot 2024-10-03 114637](https://github.com/user-attachments/assets/627f9d70-5f0b-4f68-8ed5-3fb9fd334659)


**3.** Install node.js with the command below.

```bash
sudo apt-get install nodejs -y
```
![Screenshot 2024-10-03 114810](https://github.com/user-attachments/assets/22529053-2a9a-48ba-a5c7-c2a48404b9da)

**Note:** the above command installs both node.js and npm. NPM is a package manager for Node just as apt is a package manager for Ubuntu. It is used to install Node modules and packages and to manage dependency conflicts.

**4** Verify the Node installation with the command below.

```bash
node -v        // Gives the node version

npm -v        // Gives the node package manager version
```
![Screenshot 2024-10-03 114923](https://github.com/user-attachments/assets/0fd852d1-b8be-4799-9635-f81980206f3e)

## Application Code Setup

**1**. Create a new directory for the TO-DO project and switch to it. Then initialize the project directory.
```bash
mkdir Todo && ls && cd Todo

npm init
```
![Screenshot 2024-10-03 115343](https://github.com/user-attachments/assets/60be1d65-0766-4e42-ab2a-0514ae66d712)

This is to initialize the project directory and in the process, creates a new file called package.json. This file will contain information about your application and the dependencies it needs to run.
Follow the prompts after running the command. You can press “Enter” several times to accept default values, then accept to write out the package.json file by typing yes.


## Install ExpressJs
Express is a Node.js framework that streamlines development by handling many of the lower-level tasks. It simplifies the process of defining application routes by using HTTP methods and URLs, making it easier to manage web requests and responses.

**1.** Install Express using npm

```bash
npm install express
```
![Screenshot 2024-10-03 115601](https://github.com/user-attachments/assets/4208f5f4-bab2-4d0e-ba45-f7e0a6a8c237)

**2.** Create a file index.js and run ls to confirm the file

```bash
touch index.js && ls
```
![Screenshot 2024-10-03 115601](https://github.com/user-attachments/assets/ad007cff-e12a-49a0-b4cf-e382d83a8f9a)


**3.** Install dotenv module
```bash
npm install dotenv
```
![Screenshot 2024-10-03 115637](https://github.com/user-attachments/assets/f0977621-43da-431d-a56c-a0e497e86ec2)

**4.** Open index.js file
```bash
vim index.js
```

Type the code below into it

```bash
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

app.use((req, res, next) => {
  res.send('Welcome to Express');
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```
![Screenshot 2024-10-03 115835](https://github.com/user-attachments/assets/3d956b32-74d2-4afa-bf02-ff571d28b9ab)


**Note:** Port 5000 have been specified to be used in the code. This was required later on the browser.

**5.** Start the server to see if it works. Open your terminal in the same directory as your index.js file. Run
```bash
node index.js
```
![Screenshot 2024-10-03 120510](https://github.com/user-attachments/assets/4dad148c-c2f0-49d4-af7a-91d67e15352e)


Port 5000 has been opened in ec2 security group.

Access the server with the public IP followed by the port

```bash
http://54.161.127.212:5000
```

![Screenshot 2024-10-03 120230](https://github.com/user-attachments/assets/308a9ea9-0ba8-473e-a808-124b1da6c03d)

## Routes

There are three actions that the ToDo application needs to be able to do:
- Create a new task
- Display list of all task
- Delete a completed task

Each task was associated with some particular endpoint and used different standard __HTTP__ request methods: __POST__, __GET__, __DELETE__.

For each task, routes were created which defined various endpoints that the ToDo app depends on.

**1**. Create a folder routes, switch to routes directory and create a file api.js. Open the file
```bash
mkdir routes && cd routes && touch api.js
```
![Screenshot 2024-10-03 120558](https://github.com/user-attachments/assets/3e202ddb-7ba3-4a8d-9619-2c89c6a02c07)

![Screenshot 2024-10-03 120707](https://github.com/user-attachments/assets/acb2f230-f2d3-48ee-96b7-e3dd1583c845)

__Copy__ the code below into the file

```bash
const express = require('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

});

module.exports = router;
```

![Screenshot 2024-10-03 120656](https://github.com/user-attachments/assets/0820b25a-f540-4d29-966a-055e4e574e71)

## Models

A model is at the heart of JavaScript based applications and it is what makes it interactive.

Models was used to define the database schema. This is important in order be able to define the fields stored in each Mongodb document.

In essence, the schema is a blueprint of how the database is constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties.
To create a schema and a model, mongoose  was installed, which is a Node.js package that makes working with mongodb easier.

__1.__ Change the directory back to Todo folder and install mongoose
```bash
npm install mongoose
```
![Screenshot 2024-10-03 121059](https://github.com/user-attachments/assets/e5d821ab-1dc5-4a91-a472-626d38d671a5)


**2.** Create a new folder models, switch to models directory, create a file todo.js inside models. Open the file

```bash
mkdir models && cd models && touch todo.js
```
![Screenshot 2024-10-03 121214](https://github.com/user-attachments/assets/665a221d-4ed4-4a62-99db-8dd2eeeca83f)

![Screenshot 2024-10-03 121416](https://github.com/user-attachments/assets/f200286a-7b05-4357-9871-3c3aef52f97d)

Past the code below into the file

```bash
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

// Create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```


The routes was updated from the file api.js in the ‘routes’ directory to make use of the new model.

**3**. In Routes directory, open api.js and delete the code inside with :%d_.

```bash
vim api.js
```
![Screenshot 2024-10-03 121931](https://github.com/user-attachments/assets/c31a87f1-99bd-422d-9d6b-881c0aaf0acc)

Paste the new code below into it
```bash
const express = require('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {
  // This will return all the data, exposing only the id and action field to the client
  Todo.find({}, 'action')
    .then(data => res.json(data))
    .catch(next);
});

router.post('/todos', (req, res, next) => {
  if (req.body.action) {
    Todo.create(req.body)
      .then(data => res.json(data))
      .catch(next);
  } else {
    res.json({
      error: "The input field is empty"
    });
  }
});

router.delete('/todos/:id', (req, res, next) => {
  Todo.findOneAndDelete({"_id": req.params.id})
    .then(data => res.json(data))
    .catch(next);
});

module.exports = router;
```

![Screenshot 2024-10-03 121907](https://github.com/user-attachments/assets/2322c593-d025-4192-a4d5-254b8d940263)


## MongoDB Database

**mLab** provides MongoDB database as a service solution (DBaaS). MongoDB has two cloud database management system components: mLab and Atlas, Both were formerly cloud databases managed by MongoDB (MongoDB acquired mLab in 2018, with certain differences). In November, MongoDB merged the two cloud databases and as such, __mLab.com__ redirects to the __MongoDB Atlas website__.

**1.** Create a MongoDB database and collection inside mLab.

MongoDB Cluster Overview
![Screenshot 2024-10-03 124316](https://github.com/user-attachments/assets/4880377d-c14b-42ff-b216-51f763c77d10)


AWS cloud provider, in region Aws Paris (us-west-3) was selected.

![Screenshot 2024-10-03 125106](https://github.com/user-attachments/assets/29b764b9-e70c-43da-916b-c566d97775b4)


Access from anywhere to the MongoDB database was allowed (Not secure but it is ideal for testing).

![Screenshot 2024-10-03 124421](https://github.com/user-attachments/assets/6675793e-3b75-423b-a7ff-7ee99124d3ba)


A __database__ named __todo-database__ and __collections__ named __todo_db__ was created.

![Screenshot 2024-10-03 125158](https://github.com/user-attachments/assets/d5e017d9-6480-4553-b1c0-860474d8da76)


**2.** __Create a file in your Todo directory and name it .env, open the file__
```bash
touch .env && vim .env
```

Add connection string below to access the database

```bash
DB = ‘mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority’
```
![Screenshot 2024-10-03 125325](https://github.com/user-attachments/assets/1b07a4b1-857d-4b3c-9aab-db6d4d175aa6)

![Screenshot 2024-10-03 125342](https://github.com/user-attachments/assets/7e7199ab-1bf3-4b7e-8aa2-0dc632cbcd5f)

![Screenshot 2024-10-03 125412](https://github.com/user-attachments/assets/e4cdec18-d74e-46f4-b187-83e957d9d80d)

![Screenshot 2024-10-03 125500](https://github.com/user-attachments/assets/d0828916-08d7-474e-b38c-424d7e763763)

![Screenshot 2024-10-03 125707](https://github.com/user-attachments/assets/9641437b-3e4f-4dbe-9858-b8ee3df82f51)

__3.__ __Update the index.js to reflect the use of .env so that Node.js can connect to the database__.

```bash
vim index.js
```

Delete existing content in the file, and update it with the entire code below:

```bash
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

// Connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log(`Database connected successfully`))
  .catch(err => console.log(err));

// Since mongoose promise is deprecated, we override it with Node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
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
  console.log(`Server running on port ${port}`);
});

```


Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

__4.__ Start your server using the command
```bash
node index.js
```
![Screenshot 2024-10-03 133236](https://github.com/user-attachments/assets/c19064b5-f2c9-4b12-95a4-b4b0a1d1f0bc)


There was a deprecation warning as diplayed in the image above.
To silent this warning, __{ useNewUrlParser: true, useUnifiedTopology: true }__ was removed from the code.


## Testing Backend Code without Frontend using RESTful API

Postman was used to test the backend code.
The endpoints were tested. For the endpoints that require body, JSON was sent back with the necessary fields since it’s what was set up in the code.

__1.__ Open Postman and Set the header

```bash
http://54.175.65.60:5000/api/todos
```
![Screenshot 2024-10-03 135038](https://github.com/user-attachments/assets/46fff187-d871-4924-af7e-762312767d26)

### Create POST requests to the API
![Screenshot 2024-10-03 135242](https://github.com/user-attachments/assets/2d6719fa-3b58-434f-b0c2-b1e329b9ca5b)

![Screenshot 2024-10-03 135636](https://github.com/user-attachments/assets/83d84444-411b-4857-b893-ec37f50464f2)

### Check Database Collections
![Screenshot 2024-10-03 140001](https://github.com/user-attachments/assets/6dbcea7e-0f1a-4697-9a06-a779174a676b)


### Make a GET requests to the API

This request retrieves all existing records from our To-Do application (backend requests these records from the database and sends us back as a response to GET request).

![Screenshot 2024-10-03 135716](https://github.com/user-attachments/assets/c1d243cf-5bf7-4f80-9bcc-fb8db08f747f)
![Screenshot 2024-10-03 135808](https://github.com/user-attachments/assets/7b4c4e42-d83a-4b07-bfc6-c22e02b13926)


### Create a DELETE requests to the API

![Screenshot 2024-10-03 135911](https://github.com/user-attachments/assets/1f04861f-d323-418f-af94-21f3bb99a713)

###  Check Database Collections

![Screenshot 2024-10-03 140001](https://github.com/user-attachments/assets/7dd81473-bab2-4c75-b4bb-d6139bb74a9d)


### Make another GET requests to the API

![Screenshot 2024-10-03 135925](https://github.com/user-attachments/assets/4f139635-aaab-4101-9386-64063d33293c)



## Step 2 - Frontend Creation

It is time to create a user interface for a Web client (browser) to interact with the application via API

__1.__ __In the same root directory as your backend code, which is the Todo directory, run:__

```bash
npx create-react-app client
```
![Screenshot 2024-10-03 154402](https://github.com/user-attachments/assets/fe011f84-664b-4ff4-aa60-2d906d8d8100)

if it fails, you can follow the steps below 
- i
``` bash
yarn cache clean && yarn install
```
- Then a yarn.lock file would be created, then do the next step below.
- ii
  ``` bash
  yarn add react-scripts@latest
  ```
![Screenshot 2024-10-04 104249](https://github.com/user-attachments/assets/ae8b45c9-97b9-4fd1-97ff-eb58ca796e0e)

- iii
``` bash
yarn add react@latest react-dom@latest
```
![Screenshot 2024-10-04 104309](https://github.com/user-attachments/assets/c284a68e-c132-4c0f-93ba-b85b5adda0f3)

__1.__ __In the same root directory as your backend code, which is the Todo directory, you should now run the code below again:__

```bash
npx create-react-app client
```
  
This would create a new folder in the Todo directory called client, where all the react code would be added.
![Screenshot 2024-10-04 104409](https://github.com/user-attachments/assets/1a849251-4a6e-42b4-9a42-1d750d0c3eea)

### Running a React App

Before testing the react app, the following dependencies needs to be installed in the project root directory.

- __Install concurrently__. It is used to run more than one command simultaneously from the same terminal window.
```bash
npm install concurrently --save-dev
```
![Screenshot 2024-10-03 154841](https://github.com/user-attachments/assets/62324420-29d5-4ff4-bc96-25fe86aa7317)



- __Install nodemon__. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
```bash
npm install nodemon --save-dev
```
![Screenshot 2024-10-03 163209](https://github.com/user-attachments/assets/6f7fb64f-7efb-4cd2-8891-29738321899c)

- In Todo folder open the package.json file, change the highlighted part of the below screenshot and replace with the code below:
```bash
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
}
```
![Screenshot 2024-10-03 172322](https://github.com/user-attachments/assets/b6023399-e5a9-457d-bee7-0bda573893bd)


### Configure Proxy In package.json

- Change directory to “client”
```bash
cd client
```
- Open the package.json file
```bash
vim package.json
```
![Screenshot 2024-10-04 141621](https://github.com/user-attachments/assets/5188fd1c-80e6-4745-b416-85fd90761281)

Add the key value pair in the package.json file
```bash
“proxy”: “http://localhost:5000”
```
![Screenshot 2024-10-04 141605](https://github.com/user-attachments/assets/0b8b8aab-38d8-4d51-ba95-25edbc9663da)


The whole purpose of adding the proxy configuration above is to make it possible to access the application directly from the browser by simply calling the server url like
http://locathost:5000 rather than always including the entire path like http://localhost:5000/api/todos

Ensure you are inside the Todo directory, and simply do:
```bash
npm run dev
```

The app opened and started running on localhost:3000

__Note__: In order to access the application from the internet, TCP port 3000 had been opened on EC2.
![Screenshot 2024-10-03 114116](https://github.com/user-attachments/assets/cd698e36-9b85-453e-b650-7583f0f010f1)



## Creating React Components

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For the Todo app, there are two stateful components and one stateless component. From Todo directory, run:

```bash
cd client
```

Move to the “src” directory
```bash
cd src
```
![Screenshot 2024-10-04 141741](https://github.com/user-attachments/assets/a5fb5361-81c6-4ece-bb39-20ac0ffec72e)

__2.__ __Inside your src folder, create another folder called “components”__

```bash
mkdir components
```
Move into the components directory
```bash
cd components
```
![Screenshot 2024-10-04 141741](https://github.com/user-attachments/assets/55db479d-9d55-4278-9ab8-450528752532)


__3.__ __Inside the ‘components’ directory create three files “Input.js”, “ListTodo.js” and “Todo.js”.__
```bash
touch Input.js ListTodo.js Todo.js
```
![Screenshot 2024-10-04 141751](https://github.com/user-attachments/assets/ba922ed7-457d-4440-aae2-c4b4511c4ca1)

#### Open Input.js file
```bash
vim Input.js
```
Paste in the following:

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {
  state = {
    action: ""
  }

  handleChange = (event) => {
    this.setState({ action: event.target.value });
  }

  addTodo = () => {
    const task = { action: this.state.action };

    if (task.action && task.action.length > 0) {
      axios.post('/api/todos', task)
        .then(res => {
          if (res.data) {
            this.props.getTodos();
            this.setState({ action: "" });
          }
        })
        .catch(err => console.log(err));
    } else {
      console.log('Input field required');
    }
  }

  render() {
    let { action } = this.state;
    return (
      <div>
        <input type="text" onChange={this.handleChange} value={action} />
        <button onClick={this.addTodo}>add todo</button>
      </div>
    );
  }
}

export default Input;
```
![Screenshot 2024-10-04 141823](https://github.com/user-attachments/assets/35cc9dfb-4bc1-4292-ab72-5abf2eb6c57c)


In oder to make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.

Move to the client folder
```bash
cd ../..
```
__Install Axios__
```bash
npm install axios
```


#### Go to components directory
```bash
cd src/components
```
![Screenshot 2024-10-04 142430](https://github.com/user-attachments/assets/8d249e80-b664-46dd-b884-53f6cb76352c)

#### After that open the ListTodo.js

```bash
vim ListTodo.js
```
Copy and paste the following code:

```bash
import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {
  return (
    <ul>
      {
        todos && todos.length > 0 ? (
          todos.map(todo => {
            return (
              <li key={todo._id} onClick={() => deleteTodo(todo._id)}>
                {todo.action}
              </li>
            );
          })
        ) : (
          <li>No todo(s) left</li>
        )
      }
    </ul>
  );
}

export default ListTodo;
```

![Screenshot 2024-10-04 142350](https://github.com/user-attachments/assets/ba1123ca-530d-4328-a431-1718cf50a630)

#### Then in the Todo.js file, write the following code

```bash
vim Todo.js
```

```bash
import React, { Component } from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {
  state = {
    todos: []
  }

  componentDidMount() {
    this.getTodos();
  }

  getTodos = () => {
    axios.get('/api/todos')
      .then(res => {
        if (res.data) {
          this.setState({
            todos: res.data
          });
        }
      })
      .catch(err => console.log(err));
  }

  deleteTodo = (id) => {
    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if (res.data) {
          this.getTodos();
        }
      })
      .catch(err => console.log(err));
  }

  render() {
    let { todos } = this.state;
    return (
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos} />
        <ListTodo todos={todos} deleteTodo={this.deleteTodo} />
      </div>
    );
  }
}

export default Todo;
```
![Screenshot 2024-10-04 142407](https://github.com/user-attachments/assets/0184bcd4-1719-47f9-af88-67272f13cec5)


__We need to make a little adjustment to our react code. Delete the logo and adjust our App.js to look like this__

### Move to src folder
```bash
cd ..
```

Ensure to be in the src folder and run:
```bash
vim App.js
```

#### Copy and paste the following code

```bash
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
![Screenshot 2024-10-04 142601](https://github.com/user-attachments/assets/00b64d44-aa79-4019-81f9-2721eb721c78)


####  In the src directory, open the App.css

```bash
vim App.css
```

Paste the following code into it

```bash
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
    width: 100%;
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
![Screenshot 2024-10-04 142646](https://github.com/user-attachments/assets/ecb0a511-3679-443b-a9ee-cb3a23038c1b)


#### In the src directory, open the index.css

```bash
vim index.css
```
#### Copy and paste the code below:

```bash
body {
  margin: 0;
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  background-color: #282c34;
  color: #787a80;
}

code {
  font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New", monospace;
}
```
![Screenshot 2024-10-04 142813](https://github.com/user-attachments/assets/e00dbc05-164b-4529-af75-1ac0d99eee66)
![Screenshot 2024-10-04 142828](https://github.com/user-attachments/assets/11da99eb-1278-4254-858d-f6aa9c5d5759)


#### Go to the Todo directory
```bash
cd ../..
```

Run:
```bash
npm run dev
```
![Screenshot 2024-10-04 112310](https://github.com/user-attachments/assets/b0811e2a-a4f2-4ccd-bd1c-a56da5f129af)


At this point, the To-Do app is ready and fully functional with the functionality discussed earlier: Creating a task, deleting a task, and viewing all the tasks.

__The client can now be viewed in the browser__

__Add some todos via the browser .__

__Check them on the MongoDB database__

![Screenshot 2024-10-04 140914](https://github.com/user-attachments/assets/9e75ac63-8539-4ca0-a7fd-39eee5fce317)


### Conclusion

By following this documentation and leveraging the provided resources, one is well-equipped to build and deploy full-fledged web applications utilizing the MERN stack.

Written by Arowolo-Kehinde
