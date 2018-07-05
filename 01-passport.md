(slide: Quick midterm review)

# Quick midterm review

## <%= players[i].firstName.substring(0,1) %>. <%= ... %>

## <%= `${players[i].firstName.substring(0,1)}. ${...}`%>

## <% {result=code} %> <%= result %>

## Create custom method in model

```js
playerSchema.methods.name = function() {
  return this.firstName.substring(0, 1) + '. ' + this.lastName;
};
```

```html
<td>
  <%= Model[i].name() %>
</td>
```

(slide --save-dev)

# Another tip Use --save-dev for local installation of nodemon

`npm i nodemon -D` or `npm i nodemon --save-dev`

(slide ---download 7 starter template)

# Passport Authentication

1.  Everyone [download the zip of this new summary/styling of what we've done so far](https://github.com/avcoder/lesson7starter)
1.  Remember to `npm i`
1.  Remember to include your variables.env

### Notice:

1.  It still has our Play, Admin and CRUD methods, changed Delete prompt to traditional way for now.
1.  I created nav links called "Register" and "Login" that lead to blank pages for now
1.  In app.js, for the error handling, I passed in a title as well

We'll now need to install some passport packages, then configure them in app.js
Then we'll need to deal with forms in register and login pages.
Once we can do that, we'll make changes to admin page - add conditional logic so that admin/add is hidden unless logged in
Also hide register, login. Show username.
Not only hide add/edit/delete links but Also restrict access to their respective urls

## Install 4 packages:

`npm i passport passport-local passport-local-mongoose express-session`

Examine each via https://npm.im/

1.  passport
1.  passport-local (we're using a local store of usernames and passwords)
1.  passport-local-mongoose (we're storing the users in our database and we're using mongodb. We're using mongoose to talk to db. We could have used something other than mongodb)
1.  express-session (used for managing user sessions)

## Configure app.js to use the above

3 of the 4 will go into app.js, 1 of them will go somewhere else
Then as we saw in the video, we need to set passport up

```js
/* in app.js */
/* passport deps */
const passport = require('passport');
const session = require('express-session');
```

We use the above for 2 purposes:

1.  To store user info in session object
1.  to store messages in a session so we can display that message (ex: if user puts invalid login, store message in a session first before redirecting)

```js
/* in app.js */
/* passport deps */
const localStrategy = require('passport-local').Strategy;
```

Instead of assigning all of passport-local, we only want the Strategy class. The strategy defines what type of login we're doing.
We can't configure passport until after the line `var app = express();` because we're configuring it on our express app. Since our express app does not exist until line 18? our passport configuration has to happen after this line, but above our app.use('/'...) section.

Q. Why does passport config have to happen before ou routes?
A. Because it's a middleware, it will execute first, and the routes/ctrls will use passport for securing certain pages, registering users etc. So these files need to use passport. If passport was below our routes, then passport will fail because passport hasn't been configured yet.

```js
/* passport config BEFORE controller references */
app.use(
  session({
    secret: 'some string value here',
    resave: true,
    saveUninitialized: false
  })
);
```

- May have to turn eslint features off like comma-dangle `/* eslint comma-dangle: 0, indent: 0 */`

We will pass in some configuration options:

- secret: it will be used as salt in encryption
- if user updates the page, but nothing on the page has changed, refresh session so it doesn't time out
- anonymous users on the site when they haven't logged in, don't store a session about that user because it's just overhead -- affects performance. Don't store empty sessions, only store when someone is logged in.

```js
/* initialize passport */
app.use(passport.initialize());
app.use(passport.session());
```

- We also want to tell the application to initialize passport; it has a method called initialize which we want to invoke
- Also to enable the sessions, we have to call passport's session method.

Right now we only have one model, the Game model, that describes a schema (title, year, publisher etc), that Game model is based on mongoose which talks to mongodb, so we can CRUD. But now we'll need an Account model that will be responsible for storing users in mongodb

```js
/* use the Account model to manage users */
const Account = require('./models/account');
passport.use(Account.createStrategy());
```

Now we'll create the above model. The 2nd line tells passport which model to use to save users.

```js
/* read/write user login info to mongodb */
passport.serializeUser(User.serializeUser());
passport.deserializeUser(User.deserializeUser());
```

So passport uses mongodb to keep track of a user's status by reading/writing their status to/from the db.
So this is configured now, except we still need to create the user model.

# User model

1.  Create models/User.js

Now remember we've used 3 of the 4 packages we've installed so far, until now.  
passport-local-mongoose makes our User model different from our other models.

```js
const mongoose = require('mongoose');
/* reference passport-local-mongoose to make this model usable for managing Users */
const passportLocalMongoose = require('passport-local-mongoose');
```

Our Game's model doesn't need that reference because it has nothing to do with authentication/passport. By requiring it here, we're telling passport, this is the model to use for our local user store.

```js
/* create the model schema */
/* username and password are included automatically */
const userSchema = new mongoose.Schema({});
```

So far the above is similar to our Game model. We could do that in userSchema for username/pwd but we don't have to.
Video mentioned we have helper code for that. Notice video had only nickname/birthdate but no username/pwd. He actually explained why, it's not wrong, but it's not necessary to define. Why?
Because passport just assumes that they're there already. IF we have local users, then by default we have usernames and passwords.

Let's now use our reference

```js
userSchema.plugin(passportLocalMongoose);

module.exports = mongoose.model('User', userSchema);
```

This is the final step that let's passport know, this is the model that stores User accounts.
Then we export it. Now our app should run. Configuration part is done. We've done the hard part.
Now we can start calling middleware functions.

But before we do, everyone go to mlab, and find the particular database that you're using. Run `nodemon` or `npm start` then refresh mlab and you will see a new `accounts` collection. Why?

# Create register.ejs and login.ejs

Go into Blackboard, and copy ejs code into respective ejs pages

Try logging in, it will give an error.
Go into app.js and change our error code to:

```js
res.render('error', { title: 'Retro Games' });
```

Then change our error.ejs to

```html
<% include partials/header %>

  <h1>We're sorry</h1>

  <p>Your request could not be completed. Please try again.</p>

<% include partials/footer %>
```
