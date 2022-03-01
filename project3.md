# Documentaion of Project 3

STEP 1 – BACKEND CONFIGURATION

Update ubuntu with tht command below

`sudo apt update`

Screenshot Below

![Update Ubuntu](./Images/1-Update-Ubuntu.png)

Upgrade ubuntu with the command below

`sudo apt upgrade`

Screenshot Below

![Upgrade Ubuntu](./Images/2-Upgrade-Ubuntu.png)

Getting the location of Node.js software from Ubuntu repositories using the command below

`curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -`

Screenshot Below

![curl nodejs](./Images/3-Nodejs.png)

Install Node.js with the command below

`sudo apt-get install -y nodejs`

Screenshot Below

![Install nodejs](./Images/4-Install-Nodejs.png)

Verify the node installation with the command below

`node -v`

`npm -v`

Screenshot Below

![Verify nodejs](./Images/5-Verify-Node-installation.png)

Created a new directory for your To-Do project:

`mkdir Todo`

Ran the command below to verify that the `Todo` directory is created with `ls` command

`ls`

Screenshot Below

![Make dir and list](./Images/6-Make-directory.png)

Changed current directory to the newly created one:

`cd Todo`

Used the command npm init to initialise the project, so that a new file named package.json will be created.

`npm init`

Screenshot Below

![Init](./Images/7-Run-npm-init.png)


INSTALL EXPRESSJS

Install expressjs using the npm command below

`npm install express`

Created a file index.js with the command below

`touch index.js`

Ran ls to confirm that index.js file is successfully created

Install the dotenv module

`npm install dotenv`

Screenshot Below

![Install](./Images/9-Install-Touch-list.png)

Open the index.js file with the command below

`vim index.js`

- Code pasted as shown in the screenshot below


`const express = require('express');
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
});`

![Vim](./Images/10-Vim.png)

Saved then exited

Opened terminal in the same directory as  index.js file and typed:

`node index.js`

Screenshot Below

![Node](./Images/11-Nodejs.png)

Open port 5000 in EC2 Security Groups; create an inbound rule to open TCP port 5000

Screenshot Below

![open](./Images/12-Open5000.png)

Opened browser and tried to access the server’s Public IP by port 5000: http://54.204.212.152:5000

Screenshot Below

![welcome](./Images/13-Welcome.png)

Routes

- Create a new task; POST

- Display list of all tasks; GET

- Delete a completed task; DELETE

For each task, routes were created that will define various endpoints that the To-do app will depend on. 

- Create folder routes using the command below

`mkdir routes`

- Change directory to routes folder with the command below.

`cd routes`

- Created a file api.js with the command below

`touch api.js`

See screensot below

![MKDIR](./Images/14-makedir.png)

- Opened the file with the command below 

`vim api.js`

Typed the code in the screenshot below

`const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;`

![Vim2](./Images/14-Vim2.png)

Created a Schema and a model by installing mongoose which is a Node.js package that makes working with mongodb easier.

- Changed directory back Todo folder and installed Mongoose with the command below

`npm install mongoose`

Screenshot Below
![Mongoose Install](./Images/15-Install-Mongoose.png)
- Created a new folder models, 

- Changed directory into the newly created ‘models’ folder 

- Inside the models folder, created a file and name it todo.js
 
All three commands above can be defined in one line to be executed consequently with help of && operator

`mkdir models && cd models && touch todo.js` 

Screenshot below

![Create Folders](./Images/16-createnestedfolders.png)

Opened the file created with `vim todo.js` then paste the code below in the file

`const mongoose = require('mongoose');
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

module.exports = Todo;`

Screenshot Below
![Vim3](./Images/17-Vim3.png)

Updated routes from the file api.js in ‘routes’ directory to make use of the new model.

- In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit

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

Screenshot Below

![Update Routes](./Images/18-Update-Routes.png)

- Created a MongoDB database and collection inside mLab

- Created a file in  Todo directory and name it .env. using the command below


`touch .env`
`vi .env`

- Added the connection string to access the database in it, just as below:

DB = mongodb+srv://Yetunde_Bhadmus:Sanguine123@cluster0.raxzb.mongodb.net/YetundeP3DB?retryWrites=true&w=majority

Screenshot Below

![Add Connection](./Images/18--DB.png)

- updated the index.js to reflect the use of .env so that Node.js can connect to the database i.e deleting existing content in the file, and update it with the entire code below.


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
 
Screenshot Below
![Update Vim(./Images/19-Vim4.png)

Saved then Started server using the command:

`node index.js`

see result in screenshot below

![Succesful DB](./Images/20-Database-Connected.png)

- Open  Postman, create a POST request to the API http://34.201.245.132:5000/api/todos

See Screenshots below
![Postman](./Images/21-Postman.png)

![Postman 2](./Images/22-Postman-Body.png)


Created a GET request to your API on http://34.201.245.132:5000/api/todos. This request retrieves all existing records from the To-do application

Screenshot below

![Postman3](./Images/23-getpost.png)

STEP 2 – FRONTEND CREATION

- In the same root directory as my backend code, which is the Todo directory, I ran the code below

` npx create-react-app client`

Screenshot below

![Create React app](./Images/24-Create-react-app.png)

- Install concurrently using the code below

` npm install concurrently --save-dev`

![Install Concurrently](./Images/25-Install-save-dev.png)

- Install nodemon using the code below

` npm install nodemon --save-dev`

![Install Nodemon](./Images/26-install-nodemon.png)

In the Todo folder package.json file was opened. changed were made. See screenshot below

![VI- Edit](./Images/27-edit-package.json.png)

- Change directory to ‘client’ using the command below, 

` cd client`

- Open the package.json file using the command below, 

` vi package.json`

- Added the key value pair in the package.json file "proxy": "http://localhost:5000".

![VI-Edit](./Images/28-VI.png)

- Changed Directory to back to the Todo folder

` cd ..`

![Run Dev](./Images/29-npm-rundev.png)

- Opened TCP port 3000 on EC2 by adding a new Security Group rule

See screenshot below of the web page after adding security group on EC2 instance


![React](./Images/30-Reactpage.png)


        Creating React Components

From Todo directory ran 

`cd client`

Moved to the src directory with

`cd src`

Inside the src folder another folder called components was created with

`mkdir components`

Moved into the components directory with

`cd components`

Inside ‘components’ directory I created three files Input.js, ListTodo.js and Todo.js. with

`touch Input.js ListTodo.js Todo.js`

Opened Input.js file

`vi Input.js`
 See screenshot below

![Series](./Images/31-Series.png)

Edited the following 

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

- Saved then exited

Moved back to the src folder

`cd ..`

Move to clients folder

`cd ..`

Install Axios in the client folder with the command below
 
`npm install axios`

![Series](./Images/33-Series.png)

Moved to the ‘components’ directory using the code below

`cd src/components`

 Opened ListTodo.js with

`vi ListTodo.js` 

see screenshot below

![List todo](./Images/35--VI-listtodo.png) 

Opened Todo.js file you write the following code as shown in screenshot below

![Todo](./Images/36---VI-Todo.png) 

- Made adjustment to the react code

Moved to the src folder using the code below

`cd ..`

Ran `vi App.js` then wrote the code in it as shown in the screenshot below

![App.js](./Images/37-VI--Appjs.png) 


In the src directory opened the App.css with the code below and pasted the code as shown in the screenshot below

`vi App.css`

![App.css](./Images/38-Appcss.png) 

Saved then exited

In the src directory opened the index.css with the code below

`vim index.css`

 see screenshot below

 ![Index.css](./Images/39--Index-css.png)

 Moved to the Todo directory with the code below 

 `cd ../..`
  
  Ran in the Todo directory `npm run dev`

Screenshot Below

![Index.css](./Images/40-run-dev.png)

Output on the web browser, see screen shot below

![Index.css](./Images/41-Finally.png)







 





























































