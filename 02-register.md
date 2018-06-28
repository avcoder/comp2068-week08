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
  Account.register(
    new Account({ username: req.body.username }),
    req.body.password,
    function(err, account) {
      if (err) {
        console.log(err);
        res.redirect('error', { title: 'Register Error' });
      }
      res.redirect('/login'); /* success */
    }
  );
};
```

Notice register() is a built-in method to passport-local. Also we just pass in the username. You'd think we'd also pass in the password, but no; it doesn't work like that. Rather it wants the password as a separate value. So the username is part of a JSON object, the password isn't.
Passport will salt/hash the password but it doesn't actually store the password in a property called password, it will give it a different name, in the db.

So now that we've coded above, let's go to mlab and see once we register, if a new user is created.
Q. Do you see an accounts collection? How did it get there?
A. Because once you run app.js, it initializes passport which creates this account collection in mlab if it doesn't exist.
