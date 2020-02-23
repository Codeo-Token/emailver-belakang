[Passport](http://www.passportjs.org/) is authentication middleware for [Node.js](https://nodejs.org/). As it’s extremely flexible and modular, Passport can be unobtrusively dropped in to any [Express](https://expressjs.com/)-based web application. A comprehensive set of strategies supports authentication using a [username and password](http://www.passportjs.org/docs/username-password/), [Facebook](http://www.passportjs.org/docs/facebook/), [Twitter](http://www.passportjs.org/docs/twitter/), and [more](http://www.passportjs.org/packages/).



The following packages used:

- e[xpress](https://expressjs.com/), a minimal and flexible Node.js web application framework
- b[crypt](https://github.com/kelektiv/node.bcrypt.js), A library to help you hash passwords.
- [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken), An implementation of [JSON Web Tokens](https://tools.ietf.org/html/rfc7519).
- [mongoose](https://mongoosejs.com/), Mongoose provides a straight-forward, schema-based solution to model your application data.
- [passport](http://www.passportjs.org/), an authentication middleware for [Node.js](https://nodejs.org/). Extremely flexible and modular, Passport can be unobtrusively dropped into any [Express](https://expressjs.com/)-based web application.
- [passport-jwt](http://www.passportjs.org/packages/passport-jwt/), A Passport strategy for authenticating with a JSON Web Token.
- [cors](https://github.com/expressjs/cors), CORS is a node.js package for providing a [Connect](http://www.senchalabs.org/connect/)/[Express](http://expressjs.com/) middleware that can be used to enable [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) with various options.
- [dotenv](https://github.com/motdotla/dotenv), Dotenv is a zero-dependency module that loads environment variables from a `.env` file into `process.env`.
- [express-validator](https://express-validator.github.io/docs/), express-validator is a set of [express.js](http://expressjs.com/) middlewares that wraps [validator.js](https://github.com/chriso/validator.js) validator and sanitizer functions.

**Image Upload Dependencies**

- [multer](https://github.com/expressjs/multer): Node.js middleware for handling `multipart/form-data`.
- [datauri](https://github.com/data-uri/datauri), Node.js [Module](https://www.npmjs.com/package/datauri#module) and [CLI](http://npm.im/datauri-cli) to generate [Data URI scheme](http://en.wikipedia.org/wiki/Data_URI_scheme).
- [cloudinary](https://github.com/cloudinary/cloudinary_npm)

## Middleware

```
jwt.js
The Passport JWT authentication strategy is created and configured."fromAuthHeaderAsBearerToken() creates a new extractor that looks for the JWT in the authorization header with the scheme 'bearer'".
```

```
authenticate.js
This middleware is used to authenticate the users request. Using passport.authenticate() and specifying the 'jwt' strategy the request is authenticated by checking for the standard Authorization header and verifying the verification token, if any.If unable to authenticate request, an error message is returned.
```

```
validate.js
This middleware is used to check if the express-validator middleware returns an error.If so, it recreates the error object using the param and msg keys and returns the error.
```



## utils* folder

```
index.js

Two helper functions are created in this file.

uploader

used for uploading images to cloudinary, accepts the request as a parameter.the Datauri package is used to recreate the image using the 'buffer' key in 'req.file'. The image's content is passed to cloudinary upload function. If the upload is successful, the uploaded image url is returned.----

sendEmail method

used for sending emails using sendgrid package, accepts an object containing the from, to, subject and text or html info, the object is passed to sendgrid send function to send the the email. If successful, the result is returned. ----
```



## Model

```
pre-save hook
used to hash the user’s password using the bcrypt package whenever a user is created or their password is changed before saving in the database.----

comparePassword method 
used to compare the password entered by the user during login to the user’s password currently in the database.----

generateJWT method
used for creating the authentication tokens using the jwt package. This token will be returned to the user and will be required for accessing protected routes. The token payload includes the user’s first name, last name, username and email address and is set to expire 60 days in the future.----

generatePasswordReset method 
used to generate a password reset token using the crytpo package and and calculates an expiry time (1 hour), the user object is updated with this data.----

getVerificationToken method
used to generate a token and creates and returns an instance of the Token model.
```



## Controller

```
auth.js
The authentication controller includes the following functions.register
the database is queried using the email address to check if the user already exist.If the user doesn't exist, a new user is created and saved in the database, the sendVerificationEmail function is called.--

login
the database is queried using the email address to check if the user exist.If the user is found, the user object is used to call the comparePassword method.If the comparison is successful, check if the user has been verified.if yes, the generateJWT method is called to generate the authentication token which is then passed to the user along with the user object.--

verify
using the token passed as part of the verification link, the database is queried, if the token is found, check if the user has already being verified, if not, the isVerfied field in the user's object is updated and saved.--

resendToken
the database is queried using the users email address to retrieve the users object, if found, check if the user has already being verified, if not, the sendVerificationEmail function is called.--

sendVerificationEmail
calls the getVerificationToken method, returning an instance of the Token model which is then saved in the database. A link is created and the from, to, subject and html for the email is set and passed to the sendEmail util function.
```# emailver-belakang


password.js
The password controller includes the following functions.
recover
the database is queried using the user's email address to retrieve the user's object, if found, the generatePasswordReset method is called to generate a password reset token and set its expiry time (1 hour) which is then added to the users object and saved.
A reset link is created and the from, to, subject and html for the email is set and passed to the sendEmail util function.
--
reset
queries the database for the users object using the password reset token and verifying it's still valid by adding resetPasswordExpires: {$gt: Date.now()}.
If user is found, the password reset page is displayed.
---
resetPassword
queries the database using the password reset token and veryfying it is still valid by adding resetPasswordExpires: {$gt: Date.now()}.
if the token is still valid, the users password is updated, the resetPasswordToken and resetPasswordExpires fields are set to undefined and the user object is saved and an email is sent to the user confirming the change.


user.js
update
the user id is extracted from the request params variable, there's a check to make sure the data being updated belongs to the user.
An update object is created using the request body and Mongoose findByIdAndUpdate is used to query and update the user’s data.
The req.file variable is checked to determine if an image was uploaded, if yes, the req variable is passed to the uploader util function.
If upload is succesful, the url is added to the users object and saved.


##Routes

index.js
The /api/auth uses the auth routes.
The /api/user uses the user routes. 
A middleware is attached to the user endpoint, the middleware checks and authenticates the jwt token passed before accessing the user routes.
--
auth.js
This validate middleware is used to check and validate all inputs during registration and login.
--
user.js
In this file, the multer package is used to create the middleware upload which accept a single file with the field name profileImage. 
This middleware is attached to the update endpoint to allow for multipart/form-data.

const multer = require('multer');
const upload = multer().single('profileImage');

router.put('/:id', upload, User.update);



User Index
Try accessing the user index. /api/user

![alt text](/home/thomas/Documents/verivikasi-email/1_ZG3n-ExvaKK-xwx0J6ADMg.gif)


Register and Login
Create a POST request to /api/auth/register .
Create a POST request to /api/auth/login .

![alt text](/home/thomas/Documents/verivikasi-email/1_H-xi4s-d5tH7cNXUYJYBCQ.gif)


Update User Info and Upload Profile Image

Try updating the user information and uploading a profile image using endpoint/api/user/[your_user_id] passing the token.

Login and Recover Password
Login with your current password.
Create a POST request to /api/auth/recover .

![alt text](/home/thomas/Documents/verivikasi-email/1_ncy1x2t3LELCwlS1rZZM4A.gif)


