- [ ] Show `npm i ndb`\
	- [ ] Buttons
		- [ ] Resume = Continue until next breakpoint
		- [ ] Step over a function call
		- [ ] Step into function call
	- [ ] Run script
	- [ ] Show app -> router variable -> stack
	- [ ] Show Global -> process variable
	- [ ] Set breakpoint on getAllPosts
		- [ ] show variables
## **What is a middleware**
Problem statement: Any user can delete any user's account in DB by calling `DELETE /user/:id`
We want some authentication to happen before we go ahead and delete user data.

- [ ] `next()` passes control to the next middleware function. Note: next() is having no parameters
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
  
exports.authorizeUser = async function (req, res, next) {
    const passwordInHeaders = req.headers.password;
    const { id } = req.params;
  
    // I will search the user in DB
    const user = await UserModel.findById(id);
    // Compare the DB password with the one provided in headers
    if (user.password === passwordInHeaders) {
        // if matched, go to next middleware
        next();
    } else {
        // else return 401 Unauthorized
        res.status(401).json({
            status: "fail",
            message: `You are not authorized to perform this operation`,
        });
        return;
    }
};
```
- [ ] Handling undefined routes
	- [ ] After all the routers add this
```js
app.all('*',(req,res,next){
		res.status(404).json({
		status:"fail",
		message:`Cant find ${req.originalURL} on this server`
		})
})
```
- Global error handling middleware need
	- [ ] no matter if it's an error coming from a route handler, or a model validator or really, someplace else, the goal is that all these errors end up in one central error handling middleware. So that we can send a nice response back to the client letting them know what happened.
	-  Give **4 arguments** to middleware and express will recognize it as an error-handling middleware, and thfr only call it when there is an error
    - To access the status code of the error, we will have to define it on the err.statusCode.
    - To call an error from a middleware, you need to call next() by passing err as argument
    - If you pass anything as an argument to next(), express will assume that it's an error, and will pass it to the global error handling middleware essentially **skipping** all other middlewares
```js
//How to throw error for error handling middleware
app.all('*',(req,res,next)=>{
  const err=new Error(`Cant find ${req.originalUrl} on this server!`);
  err.statusCode=404;
  err.status='fail';
  next(err);
});
//How to make error handling middleware
app.use((err,req,res,next)=>{
  err.statusCode=err.statusCode || 500;
  err.status=err.status || 'fail';

  res.status(err.statusCode).json({
    status:err.status,
    message:err.message,
  });
});
```

- [ ] Two types of errors, Operational and programming

| Operational = fail                                                 | Programming = error                                                |
| ------------------------------------------------------------------ | ------------------------------------------------------------------ |
| **Problems which we know would <br>happen and we plan in advance** | **Bugs that we devs introduce in our code<br>Difficult to handle** |
| Invalid path accessed                                              | Reading properties of undefined                                    |
| Invalid user input                                                 | Passing number where string is expected                            |
| Failed to connect to server                                        | Not using `async` on sync code                                     |
| Request timeout etc.                                               | etc.                                                               |
- [ ] Create class which handles all the Operational Erros
```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);

    this.statusCode = statusCode;
    this.status = statusCode === 500 ? 'error' : 'fail';

    Error.captureStackTrace(this, this.constructor);
  }
}
```
usage - 
```js
next(new AppError(`Cant find ${req.originalUrl} on this server!`, 404));
```

- [ ] Problem with try/catch blocks in controllers
	- [ ] Handling Async controller errors
```js
To catch errors in async functions, just wrap the function inside another function which returns a function as a value, and catching the error is handled by it.

const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

exports.addTour = catchAsync(async (req, res) => {
  const newTour = await Tour.create(req.body);

  res.status(201).json({
    status: 'success',
    data: {
      tour: newTour,
    },
  });
});

In actual code, catchAsync function has been transferred to utils/catchAsync.js
```

- Handling 404 errors
```js
if (!blogPost) {
	// Return is important or else your code will try to send 2 responses
	return next(new AppError("Cant find blogpost with given id: " + id, 404));
}
```
