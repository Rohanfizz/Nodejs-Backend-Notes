- ## Request Parts
### 1. Headers

Headers in an HTTP request provide essential information about the request or the client itself to the server. They are key-value pairs that communicate metadata about the request. Reasons for including headers:

- **Content-Type**: Specifies the media type of the resource or data being sent by the request, such as `text/html`, `application/json`, etc.
- **Authorization**: Contains credentials for authenticating the client to the server, such as tokens or username/password.
- **User-Agent**: Identifies the client software making the request to the server, allowing the server to tailor responses for compatibility.
- **Accept**: Tells the server what content types the client can process, enabling content negotiation.
- **Cookies**: Sent in the header for state management, tracking session data, or maintaining user preferences.

### 2. Body

The body of an HTTP request is used to send data to the server. This part of the request is optional and is only used with methods that submit data to the server, like `POST`, `PUT`, or `PATCH`. The body is important for:

- **Submitting form data**: When a user fills out a form on a webpage and submits it, the form data is sent in the request body.
- **Uploading a file**: Files are transmitted to the server in the request body.
- **API requests**: When interacting with APIs, the request body often contains data in JSON or XML format that you're sending to the server.

### 3. Cookies

Cookies are small pieces of data stored on the client side and are sent to the server with HTTP requests through the `Cookie` header. They are used for:

- **Session management**: Identifying user sessions on the server to associate requests with the correct user data.
- **Personalization**: Storing user preferences to customize the user experience on the site.
- **Tracking**: Monitoring and analyzing user behavior on a website.

Cookies are a critical part of web functionality, enabling stateful sessions in the stateless protocol of HTTP. They ensure that users can have personalized and interactive experiences on the web.

- JWT token should be sent in the headers and not the body as headers contain data *about the client*
## Protected Route
- Uses
	- User for authorization.
	- It "Protects" a sensitive resource. Basically validates the token sent in the request headers
	- Authorization token is sent in headers of auth with, Authorization: "bearer xyztoken"
	- 4 Steps to authorize access using protect middleware
		1) add a authController.protect middleware before each protected call In protect function
		2) check if token is provided
```js
let token;

if(req.headers.authorization && req.headers.authorization.startsWith('Bearer')){
	token = req.headers.authorization.split(' ')[1];
}
if(!token){
    return next(new AppError('You are not logged in!',401));
}
```
3) Check if user still exists.
	- For this we will need to decode the token
	- `const decoded = await promisify(jwt.verify)(token, process.env.JWT_SECRET);`
	- Now using this decoded information, we can simple check if user with that id exists in the db
```js
    const currentUser = await User.findById(decoded.id);
    if (!currentUser) {
        return next(
            new AppError(
                "The user belonging to this token does no longer exist.",
                401
            )
        );
    }
```
4) Grant Access to the user
```js
    req.user = currentUser;
    res.locals.user = currentUser;
    next();
```

All the above code will together make the protect middleware
### Handle bad token, expired token
We need to now handle cases like JWT is invalid OR JWT expired
For that in our global errorController, we need to add
```js
if (error.name === "JsonWebTokenError") error = handleJWTError();
if (error.name === "TokenExpiredError") error = handleJWTExpiredError();
```
also we need to create these functions
```js
const handleJWTError = () =>
    new AppError(`Invalid Token, Please log in again`, 401);

const handleJWTExpiredError = () =>
    new AppError("Your token has been expired! Please log in again", 401);
```

### Handle changed password token access
For this we need to keep a track on when the password was changed last ( Timestamp )
- We will create a field in userSchema - `passwordChangedAt:{type:Date,defalt:Now}`
Now create the instance method,
```js
userSchema.methods.changedPasswordAfter = function (JWTTimestamp) {
		const timeStampInMiliSeconds = this.passwordChangedAt.getTime(); 
		const changedTimestamp = parseInt(// here we are converting ms to s
		timeStampInMiliseconds / 1000,
			10// base 10
	);
	return JWTTimestamp < changedTimestamp;
};
```
Now call this function in the last step of protect function
```js
// 4) Check if user changed password after the token was issued
    if (currentUser.changedPasswordAfter(decoded.iat)) {
        return next(
            new AppError(
                "User recently changed password! Please log in again.",
                401
            )
        );
    }
```

### Advance Postman Setup
- Environment variables
- Test Scripts - `pm.environment.set("jwt", pm.response.json().token);` - Used to set jwt token automatically
- instead of bearer token in headers, we are going to use `Authorization` tab in postman to have a bearer token, and specify the token in any protected route as {{jwt}}
### Forgot Password
We are going to create a temporary token, send one to user in their email, and save an encrypted in our DB

STEPS
1) generate a new token and send it to user by mail,
2) save that new unencrypted token in database after encryption for safety

  
Added to user model =>
```js
passwordResetToken: String,//in schema
passwordResetExpires: Date,//in schema
```
Additionally we are going to user  an instance method which generates a passwordResetToken
```js
userSchema.methods.createPasswordResetToken = function () {
  const resetToken = crypto.randomBytes(32).toString('hex');  // generate token

  this.passwordResetToken = crypto  
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');
  this.passwordResetExpires = Date.now() + 10 * 60 * 1000; // 10 min

  return resetToken;
};
```
controller function, (incomplete)
```js
exports.forgotPassword = catchAsync(async (req,res,next)=>{
  //  1) Get User based on Posted email
  const user = await User.findOne({email:req.body.email});
  if(!user){
    return next(new AppError('There is no user with email address.',404));
  }
  // 2) Generate random test token
  const resetToken = user.createPasswordResetToken();//use the instance method
  await user.save({validateBeforeSave: false});// Validation is turned as false because user just provides his/her email in forgetpassword, so validation like required fields will be ignored while saving the document by mongoose
})
```
- Moving forward in this controller function, we need to send this resetToken to the user as well in their email
- For that we need `nodemailer` library to send emails ->  `npm i nodemailer`
- We can configure Gmail with nodemailer, but it is not adviced, [This is why](https://nodemailer.com/usage/using-gmail/)
- Instead we are going to use mailtrap for sending emails
```js
const nodemailer = require('nodemailer');

const sendEmail = async (options) => {
  // 1) Create trasporter
  const transporter = nodemailer.createTransport({
    host: process.env.EMAIL_HOST,
    port: process.env.EMAIL_PORT,
    secure: false,
    logger: true,
    auth: {
      user: process.env.EMAIL_USERNAME,
      pass: process.env.EMAIL_PASSWORD,
    },
});

  // 2) DEFINE EMAIL OPTIONS
  const mailOptions = {
    from: 'Rohan Sharma',
    to: options.email,
    subject: options.subject,
    text: options.message,
    // html:
  };

  //   3) Actually send the email
  console.log("now sending");
  await transporter.sendMail(mailOptions);
  console.log("sent");
};

module.exports = sendEmail;
```
Now in our controller, we have to create a resetpassword URL
```js
exports.forgotPassword = catchAsync(async (req,res,next)=>{
  //  1) Get User based on Posted email
  const user = await User.findOne({email:req.body.email});
  if(!user){
    return next(new AppError('There is no user with email address.',404));
  }
  // 2) Generate random test token
  const resetToken = user.createPasswordResetToken();//use the instance method
  await user.save({validateBeforeSave: false});// Validation is turned as false because user just provides his/her email in forgetpassword, so validation like required fields will be ignored while saving the document by mongoose
  const resetURL = `${req.protocol}://${req.get('host')}/api/users/resetPassword/${resetToken}`
  const message = `Submit a patch request to this URL - ${resetURL}`
  
  await sendEmail({
	  email:user.email,
	  subject:'Your password reset token',
	  message
  });
  
  res.status(200).json({
	  status:'success',
	  message: 'Token sent to mail successfully!'
  })
})
```
- But this creates a potential flaw, if for example sending mail fails, we need to reset the passwordResetToken and passwordResetExpires field in our mongodb database, as user might try again.
- We cant simply send error response
```js
try{
 await sendEmail({
	  email:user.email,
	  subject:'Your password reset token',
	  message
  });
}catch(err){
	user.passwordResetToken = undefined;
	user.passwordResetExpires = undefined;
	await user.save({validateBeforeSave: false})
	return next(new AppError('There was a problem sendiing the email. Please try again',500))
}
```
- now reset password middleware
```js.
exports.resetPassword = catchAsync(async (req,res,next)=>{
//1) get the token from request
let token = req.param.token
//2) hash the token
 let hashedToken = crypto  
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');

//3) find the user with token in the db, also validate passwordResetExpires is $gt Date.now()

//4) if the token is not expired then user is there, set the new passowrd, passwordConfirm, set passwrodResetToken and expires to undefined as well

//5) Update changed password at property of the user

//6) log the user in and send jwt as response
})
```