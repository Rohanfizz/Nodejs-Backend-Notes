## Introduction to MongoDB

**What is MongoDB?**

- MongoDB is a NoSQL database that stores data in a flexible, JSON-like format called BSON.
- It is designed to handle large amounts of data and scale horizontally.

**Features and Benefits:**

- Schema-less database
- High performance
- Scalability
- Rich query language

**SQL vs. NoSQL:**

- SQL databases use structured query language and have a predefined schema.
- NoSQL databases like MongoDB are more flexible, allowing for unstructured data storage.
---

## Setup mongodb
- Go to https://www.mongodb.com/
- Sign in
- Click on **create** (create new cluster)
- Select M0 template (free)
- Click on Create Deployment (Do not mess with any other settings)
- Go to "Network Access" on the main page
- Provide `0.0.0.0/0` this IP and confirm

---

## Connecting MongoDB to an Express.js Application

**Setting Up Express.js Project:**

**Installing Mongoose:**
```bash
npm install mongoose
```
`

**Connecting to MongoDB:** 
- Get the DB url first
	- Click on "Connect" and go to drivers. Copy the URL for connection.
- Get the password for your URL
	- Go to "Database access" click "edit" on your username
	- Click "Edit password", click "auto generate"
	- Copy the generated password.
	- Click on Update User
	- Paste the password in url in place of `<password>`

```js
const mongoose = require('mongoose');

const DBLink = process.env.DB.replace("<password>", process.env.DB_PASSWORD);

const DB = mongoose.connect(DBLink).then(() => {
    console.log("Connected to DB ♥");
});
```
- For sensitive information like passwords, DB URL you don't want them to be included in your code. But we want to use it in the code...
- We will use environment variables to use sensitive info in the code.

Note: DB and DB_PASSWORD are defined in .env file.
### Access properties from .env files
```js
	const dotenv = require("dotenv");// Add this to the top of the application
	dotenv.config({ path: "./config.env" });
```

---
## Defining Mongoose Schemas and Models

**Creating a Schema and Model:**

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  userName: {
        type: String,
        required: [true, "Please provide a userName."],
        unique: true
    },
  age: Number,
  email: String
});

const User = mongoose.model('User', userSchema);
module.exports = User
```

**Basic Schema Types:**
- String
- Number
- Date
- Boolean
---
## Performing CRUD Operations with Mongoose (Controllers)

**Create:**
```js
exports.createUser = async function (req, res) {
    const { username, password, email } = req.body;
    try {
        await UserModel.create({ username, password, email });
    } catch (err) {
        res.status(500).send({
            status: "failure",
            error: err.message,
        });
    }
  
    res.status(200).send({
        status: "success",
        data: "User created successfully!",
    });
};

```
**Get User By Id:**
```js
exports.getUserbyId = async function (req, res) {
    const { id } = req.params;
    let user;
    try {
        user = await UserModel.findById(id);
    } catch (err) {
        res.status(500).json({
            status: "fail",
            message: err.message,
        });
        return;
    }
    res.status(200).json({
        status: "success",
        data: user,
    });
};

```

**Update User By Id:**
```js
exports.updateUserById = async function (req, res) {
    const { id } = req.params;
    const { username, email, password, role } = req.body;
    let newuser;
    try {
        newuser = await UserModel.findByIdAndUpdate(
            id,
            { username, email, password, role },
            { new: true }
        );
    } catch (err) {
        res.status(500).json({
            status: "fail",
            message: err.message,
        });
        return;
    }
    res.status(200).json({
        status: "success",
        data: newuser,
    });
};

```
**Delete User By Id:**
```js
exports.deleteUserById = async function (req, res) {
    const { id } = req.params;
    try {
        await UserModel.findByIdAndDelete(id);
    } catch (err) {
        res.status(500).json({
            status: "fail",
            message: err.message,
        });
        return;
    }
    res.status(200).json({
        status: "success",
        data: `User with id: ${id} deleted successfully`,
    });
};
```

---
### [BONUS] Connect mongoDB to mongoDB compass
- Sign in to your MongoDB Atlas account.
- Navigate to your cluster and click on the "Connect" button.
- Choose "Connect with MongoDB Compass" and copy the provided connection string.
- Paste the connection string into MongoDB Compass and replace `<password>` with your actual password.
- Click the "Connect" button.
---
## MongoDB Compass
A tool to access mongodb cluusters from your local machine. You can perform queries on the DB

## Node debugger (ndb)
- It helps you to add breakpoints and iterate line-by-line on the code. If helps in figuring out where things are going wrong.
```bash
npm i -g ndb
ndb server.js
```
