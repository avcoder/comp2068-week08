# Register

So passport is set up, let's use it.
Need to write a post handler so we send that info to passport which will generate a user account.

```js
/* in index.js */
router.post('/register', userController.register);
```

```js
/* adding ref's for signup and login
/* in userController.js */
let passport = require('passport');
let Account = require('../models/account');

exports.register = (req, res, next) => {
  const user = new Account({ username: req.body.username });
  Account.register(user, req.body.password, function(err, account) {
    if (err) {
      console.log(err);
      res.redirect('error', { title: 'Register Error' });
    }
    res.redirect('/login'); /* success */
  });
};
```

Notice register() is a built-in method to passport-local.
Passport will salt/hash the password but it doesn't actually store the password in a property called password, it will give it a different name, in the db.

So now that we've coded above, let's go to mlab and see once we register, if a new user is created.
Q. Do you see an accounts collection? How did it get there?
A. Because once you run app.js, it initializes passport which creates this account collection in mlab if it doesn't exist.

### Register now

Now go ahead and register via form and see if it goes into mlab.

Examine the new document in accounts. Notice there's no password field, just hash.

### Register with same password

Let's try registering another account but using the same password, and compare hash values.

### Register with same username

Now let's try registering another account but using the same username/email.
So passport will automatically prevent duplicate usernames.

# Login

We just need to pass in 3 values to authenticate.

```js
/* in index.js */
/* POST login */
router.post(
  '/login',
  passport.authenticate('local', {
    successRedirect: '/admin',
    failureRedirect: '/login',
    failureMessage: 'Invalid Login'
  })
);
```

### Login now

If you try entering an invalid username, there's no message yet. So we'll have to put some code to tell user what's happening.
If you put in a valid account, it takes you to admin. You have an active session.

### display message

So we should add some code to our router.get('/login'...) in case it fails

```js
/* in controller */
const messages = req.session.messages || [];

res.render('login', {
  title: 'Login',
  messages: messages
});
```

In login.ejs, before form tag

```html
<% if (messages.length > 0) { %>
  <div class="alert alert-danger"><%= messages %></div>
<% } else { %>
  <div class="alert alert-info">Please enter your credentials</div>
<% } %>
```

Above works so far, but there is a problem. If you click your Games link, then click Login again, you should see Invalid Login still.

```js
/* clear sessions messages */
/* beneath our const messages line */
req.session.messages = [];
```

So now if you logging in invalidly, then click Games, then click Login again, the session variable should be clear, so the else message appears.

Now the above trick can be used via connect-flash which saves us from clearing the session since it will automatically do that for us.

### Modify header to hide if not logged in

In games Controller

```js
res.render('games', {
  title: 'All Games'
  games,
  user: req.user
})
```

The user object is stored automatically by Express.

In header.ejs, add if statements around Admin link

```html
<% if (user) { %>
   /* display Admin link */
<% } %>
```

Refresh page to test. Then login and refresh to test.

Problem, the admin/edit/delete/add URLs still publicly exists so we have to protect those routes

## Protect routes

Create new authcontroller.js

```js
const passport = require('passport');
const mongoose = require('mongoose');
const User = mongoose.model('Account');

exports.isLoggedIn = (req, res, next) => {
  // first check if the user is authenticated
  if (req.isAuthenticated()) {
    next();
    return;
  }
  res.redirect('/login');
};
```

In routes/index.js

```js
const authController = require('../controllers/authController');
router.get('/admin', authController.isLoggedIn, gamesController.admin);
```

Also include even router.post() since user could create their own forms and just substitute action/post
Do it even for the `/play` route forcing users to login if they want to play

## Logout

```js
router.get('/logout', (req, res) => {
  req.logout();
  res.redirect('/games');
});
```

```html
<% if (!user) { %>
  /* Show Register and Login */
<% } else { %>
  /* Show Logout and username to the right of that */
<% } %>
```

## Display username beside logout
