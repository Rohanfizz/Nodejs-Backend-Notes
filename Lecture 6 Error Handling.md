- #### Production and dev envs
	- Handling errors differently
```js
module.exports = (err, req, res, next) => {
    err.statusCode = err.statusCode || 500;
    err.status = err.status || "error";
    if (process.env.NODE_ENV === "development") {
        serErrDev(err, res);
    } else {
        sendErrorProd(error, res);
    }
};
```
- For development env
```js
const serErrDev = (err, res) => {
    // since dev mode, send everything as it is
    res.status(err.statusCode).json({
        status: err.status,
        error: err,
        message: err.message,
        stack: err.stack,
    });
};
```
- For Production
```js
exports.sendErrProd = (err, res) => {
    console.log(err)
    if (err.statusCode == 500) { // Programatic error = we want to hid information
        res.status(err.statusCode).json({
            status: err.status,
            message: "Oh something bad happened!",
        });
        return;
    } else {    // Operation Error => we want to show the user what bad happened
        res.status(err.statusCode).json({
	            status: err.status,
            message: err.message,
        });
    }
};
```

- #### General types of errors in mongoDB
	- Some errors are created by mongoose like validation errors,duplication errors etc. Although these errors are operational and we have to prepare for them, initially our app treats them as programatical errors. So we have to change some stuff in the production block so as to provide user with an operational error which is provided to us by DB
	- **CastError** - Accessing getByID by passing bad ID `/wwwww`
	```js
// Add this line before calling sendErrorProd
let error = { ...err };
console.log("💥💥💥💥💥", err.name);
if (err.name === "CastError") error = handleCastErrorDB(error);
//Function to handle Cast Error
const handleCastErrorDB = (err) => {
    const message = `Invalid ${err.path}:${err.value}`;
    return new AppError(message, 400);
};
```

	- Duplicate Fields error- Creating a document with a same field which is supposed to be unique
	- Handling Duplicate Database fields.
```js
const handlerDuplicateFieldsDB = (err) => {
  // const value = err.errmsg.match(/(["'])(?:(?=(\\?))\2.)*?\1/);
  const value = err.keyValue.name;
  const message = `Duplicate field value: ${value}. Please use another Value!`;
  return new AppError(message, 400);
};
```

- Add this line too in distinguisher function
```js
if (err.code === 11000) error = handlerDuplicateFieldsDB(error);
```

- Validation errors - length is smaller OR invalid datatype passed
```js
if (err.name === 'ValidationError') error = handleValidationDB(error);
```

- Handler
```js
const handleValidationDB = (err) => {
   const errors = Object.values(err.errors).map((el) => el.message);
   const message = `Invalid input data. ${errors.join('. ')}`;
   return new AppError(message, 400);
};
```
- #### Handling all the unhandled Rejections
	- Handling all the unhandled errors even outside our express application
```js
//server.js
process.on("unhandledRejection", (err) => {
    console.log(err.name, err.message);
    process.exit(1);
});
```
For graceful exit
```js
//server.js
process.on("unhandledRejection", (err) => {
    console.log(err.name, err.message);
    server.close(()=>{
		process.exit(1);
    })
});

```
- #### Handling all the uncaughtExceptions
	- This handles all the other non-operational errors
	- for example `console.log(x)` in a controller where x is undefined
```js
//server.js before requiring application
process.on("uncaughtExceptions", (err) => {
    console.log(err.name, err.message);
	process.exit(1);
});
```

- Unit testing with Jest