- [ ] User Model + Validation on password confirm
	- [ ] Sometimes unique : true doesnt work, add   `autoIndex: true`

```js
const userSchema = new mongoose.Schema(
    {
        ...essentialSchema.obj,
        firstName: {
            type: String,
            required: [true, "Please tell us your first name!"],
        },
        lastName: {
            type: String,
        },
        email: {
            type: String,
            required: [true, "Please provide us your email"],
            unique: true,
            lowercase: true,
            validate: [validator.isEmail, "Please provide a valid email"],
        },
        photo: String,
        role: {
            type: String,
            enum: ["user", "seller", "admin"],
            default: "user",
        },
        password: {
            type: String,
            required: [true, "Please provide a password"],
            minlength: 8,
            select: false,
        },
        passwordConfirm: {
            type: String,
            required: [true, "Please confirm your password"],
            select: false,
            validate: {
            // validator only works on "save", not on findOneAndUpdate
            // we cannot use => function because we need "this keyword here"
	                validator: function (el) {
                    return el === this.password;
                },
                message: "Passwords are not the same!",
            },
        },
    }
)
```
- [ ] Route and controller
```js
exports.signup = catchAsync(async (req, res, next) => {
  const newUser = await User.create(req.body);

  res.status(201).json({
    status: 'success',
    data: {
      user: newUser
    }
  });
});

```
- [ ] Encrypting passwords
![[password.drawio.png]]
	- [ ] pre middleware by mongoose
	- [ ] bcrypt
```js
userSchema.pre('save', async function (next) {
  // Only run this function if password was actually modified
  if (!this.isModified('password')) return;

  // encrypt password with cost of 12
  this.password = await bcrypt.hash(this.password, 12);

  // Delete password confirmation field
  this.passwordConfirm = undefined;
});
```

- [ ] JWT
	- [ ] JSON web token helps maintain the law of restful APIs which says that the server should be stateless.
		Its like a passport which it has to show in order to access a protected resource.
		It is made up of 3 parts
		1. Header
		2. Payload
		3. Signature.

  

Header and payload are encoded and not encrypted which means anyone can decode them. but verification signature cannot be altered as it is made up of header,payload and signature.


## Establishing Identity With jWT

to verify if a JWT is valid and has not been altered, JWT takes in header and payload and creates a test signature with the original signature. If the test signature and the original signature (original signature is stored on server) match, access granted, else no.
![[Pasted image 20240217183610.png]]
JWT Signature verification
![[diagram.png]]
### **Authentication** -
⚠ Its a real responsibility, No mistakes are allowed here. No bugs should be present and all edge cases should be considered as user's data is at stake.

- Anyone can come and register themselves as user, so fix the contoller
```js
const newUser = await User.create({
  name: req.body.name,
  email: req.body.email,
  password: req.body.password,
  passwordConfirm: req.body.passwordConfirm
});
```
- Sign in the user as soon as they log in ? (Return a JWT token on signup)
	- `npm i jsonwebtoken` [Docs](https://github.com/auth0/node-jsonwebtoken)
```js
exports.signup = catchAsync(async (req, res, next) => {
  const newUser = await User.create(req.body);

  const token = jwt.sign({ id: newUser._id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN
  });

  res.status(201).json({
    status: 'success',
    token,
    data: {
      user: newUser
    }
  });
});

```
- #### Login
```js
exports.login = catchAsync(async (req, res, next) => {
  const { email, password } = req.body;

  // 1) Check if email and password exist
  if (!email || !password) {
    return next(new AppError('Please provide email and password!', 400));
  }

  // 2) Check if user exists && password is correct
  const user = await User.findOne({ email }).select('+password');
	// instance method used below------------------
  if (!user || !(await user.correctPassword(password, user.password))) {
    return next(new AppError('Incorrect email or password', 401));
  }

  // 3) If everything ok, send token to client
  const token = signToken(user._id); // Token should be assigned here after generation
  res.status(200).json({
    status: 'success',
    token,
  });
});
```

Instance Method -> An instance method is **a method that belongs to instances of a class, not to the class itself. It should be defined in corresponding model file**

```js
userSchema.methods.correctPassword = async function (      // instance method
  candidatePassword,
  userPassword
) {
  return await bcrypt.compare(candidatePassword, userPassword);
};
```
