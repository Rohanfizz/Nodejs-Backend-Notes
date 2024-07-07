## NPM
- Node Package Manager.
- Nodejs has a lot of packages.
- It helps you to install, uninstall, update
- NPM creates a package.json. it contains info about packages (versions, list of all packages) used by your project.


- To get started, we first need to initialize npm in the package, and install express.
```bash
npm init
npm install express
```
- Now we can setup the server using express
```node
const express = require("express")
const app = express();
app.use(express.json());    // to access body in request
const port = 8080;
const server = app.listen(port, ()=>{
	console.log(`Server is listening on port ${port}`)
})
```
- We need to setup routes so that we can test our server.
## Routes and routing

### Routes and Routing in Express.js

**Routes** define the endpoints (URIs) that your application will respond to, and they determine what code will execute when those endpoints are hit.

**Routing** refers to how an application’s endpoints (URIs) respond to client requests. Express.js provides a robust set of features for routing, which makes it easy to handle different HTTP methods and paths.

---


### Setting Up Routes

Here's an example of setting up a basic route in Express.js:

```javascript
app.get('/', (req, res) => {
    res.send('Hello World!');
});
```

This route listens for GET requests on the root URL (`/`). When a request is made to this URL, it sends back "Hello World!".

You can define routes for different HTTP methods (GET, POST, PUT, DELETE, etc.):

```javascript
app.post('/submit', (req, res) => {
    res.send('Data received');
});

app.put('/update', (req, res) => {
    res.send('Data updated');
});

app.delete('/delete', (req, res) => {
    res.send('Data deleted');
});
```

---

### Controllers in Express.js

**Controllers** in Express.js are responsible for handling incoming requests and sending responses back to the client. They help organize your application logic, making it easier to manage and scale.

#### Why Use Controllers?

- **Separation of Concerns:** By separating route definitions and business logic, your code becomes more organized and easier to maintain.
- **Reusability:** Controllers can be reused across different routes.
- **Testability:** Controllers can be tested independently from the rest of the application.

#### Setting Up Controllers

Let's create a simple controller to handle basic user operations like getting user details, creating a user, updating a user, and deleting a user.

**Step 1: Create the Controller File**

First, create a directory named `controllers` in your project root, then create a file named `userController.js` inside this directory.

**Example: userController.js**

```javascript
// Define controller functions
exports.getUser = (req, res) => {
    res.send('User details');
};

exports.createUser = (req, res) => {
    res.send('User created');
};

exports.updateUser = (req, res) => {
    res.send('User updated');
};

exports.deleteUser = (req, res) => {
    res.send('User deleted');
};
```

In this file, we define four functions (`getUser`, `createUser`, `updateUser`, `deleteUser`) to handle different user-related operations. Each function takes `req` (the request object) and `res` (the response object) as parameters and sends a response back to the client.

**Step 2: Define Routes Using Controllers**

Next, create a directory named `routes` in your project root, then create a file named `userRoutes.js` inside this directory.

**Example: userRoutes.js**

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

// Define routes and associate them with controller functions
router.get('/user', userController.getUser);
router.post('/user', userController.createUser);
router.put('/user', userController.updateUser);
router.delete('/user', userController.deleteUser);

module.exports = router;
```

In this file, we define routes and associate them with the controller functions we created earlier. Each route handles a different HTTP method (GET, POST, PUT, DELETE) and URL path.

**Step 3: Use Routes in Your Server**

Finally, modify your main server file to use the routes defined in `userRoutes.js`.

**Example: server.js**

```javascript
const express = require('express');
const app = express();
const port = 8080;

const userRoutes = require('./routes/userRoutes');

// Use the routes defined in userRoutes.js
app.use('/api', userRoutes);

const server = app.listen(port, () => {
    console.log(`Server is listening on port ${port}`);
});
```

In this file, we import the routes from `userRoutes.js` and use them in our application. The `/api` prefix is added to all routes defined in `userRoutes.js`.

---

### Middleware and the "next" Keyword in Express.js

Middleware functions are functions that have access to the request object (`req`), the response object (`res`), and the next middleware function in the application’s request-response cycle. The `next` keyword is used to pass control to the next middleware function.

#### Why Use Middleware?

- **Preprocessing Requests:** Modify or validate request data before it reaches the route handler.
- **Authentication and Authorization:** Verify user credentials before allowing access to certain routes.
- **Logging and Monitoring:** Log requests or perform monitoring tasks.

#### Using the "next" Keyword

Let's create a middleware function that checks if a user is authenticated before allowing access to certain routes.

**Example: Middleware for User Authentication**

First, create a directory named `middleware` in your project root, then create a file named `authMiddleware.js` inside this directory.

**Example: authMiddleware.js**

```javascript
// Middleware function to check if the user is authenticated
function isAuthenticated(req, res, next) {
    const userIsAuthenticated = true; // This is a placeholder. You would normally check session or token.

    if (userIsAuthenticated) {
        next(); // Pass control to the next middleware or route handler
    } else {
        res.status(401).send('User not authenticated');
    }
}

module.exports = isAuthenticated;
```

In this middleware function, we check if the user is authenticated. If they are, we call `next()` to pass control to the next middleware function or route handler. If they are not authenticated, we send a 401 status response.

**Using Middleware in Routes**

Now, use this middleware in your routes to protect them.

**Example: userRoutes.js**

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const isAuthenticated = require('../middleware/authMiddleware');

// Protect routes using the isAuthenticated middleware
router.get('/user', isAuthenticated, userController.getUser);
router.post('/user', isAuthenticated, userController.createUser);
router.put('/user', isAuthenticated, userController.updateUser);
router.delete('/user', isAuthenticated, userController.deleteUser);

module.exports = router;
```

In this file, we add the `isAuthenticated` middleware to the routes we want to protect. The middleware function runs before the controller functions, ensuring that only authenticated users can access these routes.

**Using Middleware in Your Server**

Ensure your middleware is used in the main server file if needed globally.

**Example: server.js**

```javascript
const express = require('express');
const app = express();
const port = 8080;

const userRoutes = require('./routes/userRoutes');

// Use middleware globally (if needed)
// app.use(isAuthenticated);

app.use('/api', userRoutes);

const server = app.listen(port, () => {
    console.log(`Server is listening on port ${port}`);
});
```
