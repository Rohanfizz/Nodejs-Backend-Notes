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
## Connecting MongoDB to an Express.js Application

**Setting Up Express.js Project:**

**Installing Mongoose:**
```bash
npm install mongoose
```
`

**Connecting to MongoDB:** 

```js
const mongoose = require('mongoose');

const DBLink = process.env.DB.replace("<password>", process.env.DB_PASSWORD);

const DB = mongoose.connect(DBLink).then(() => {

    console.log("Connected to DB ♥");

});
```
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
## Performing CRUD Operations with Mongoose

**Create:**
```js
const user = new User({ name: 'John', age: 30, email: 'john@example.com' });
user.save();

```
**Read:**
```js
User.find({}, function(err, users) {
  console.log(users);
});

```

**Update:**
```js
User.updateOne({ name: 'John' }, { age: 31 }, function(err) {
  if (err) console.error(err);
});

```
**Delete:**
```js
User.deleteOne({ name: 'John' }, function(err) {
  if (err) console.error(err);
});
```

---
### [BONUS] Connect mongoDB to mongoDB compass
- Sign in to your MongoDB Atlas account.
- Navigate to your cluster and click on the "Connect" button.
- Choose "Connect with MongoDB Compass" and copy the provided connection string.
- Paste the connection string into MongoDB Compass and replace `<password>` with your actual password.
- Click the "Connect" button.