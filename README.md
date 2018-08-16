# nodejs-passport-google-oauth2

This repo is based on an early commit of brillian project by Stephen Grider. The code is from a course called "Node with React: Fullstack Web Development"

https://www.udemy.com/node-with-react-fullstack-web-development/

This current version doesn't have react yet, but it contains everything you need to authenticate with Google Oauth 2.0 using NodeJS and MongoDB. The final repo by Grider <a href="https://github.com/StephenGrider/FullstackReactCode/tree/master/server">can be found here</a>

<h5>Getting Started</h5>

1. Clone the repo.

2. npm install

3. npm run dev

4. navigate to 'http://localhost:5000/auth/google'. The server redirects you to the Google Oauth page (based on the Google App's Client_ID and Client_Secret you created - see more below). Once you provide your Google Email and Password, you will be redirected to 'http://localhost:5000/auth/google/callback?code='.

The code passed within the URL will be used in passport's flow. Here's the google strategy from the <a href="services/passport.js">authRoutes.js file</a>. It contains a callback function which is executed once Google receives the Email and Password from the user, and the user submits the form.

```javascript
passport.use(
  new GoogleStrategy(
    {
      clientID: keys.googleClientID,
      clientSecret: keys.googleClientSecret,
      callbackURL: '/auth/google/callback',
      proxy: true
    },
    (accessToken, refreshToken, profile, done) => {
      User.findOne({ googleId: profile.id }).then(existingUser => {
        if (existingUser) {
          // we already have a record with the given profile ID
          done(null, existingUser);
        } else {
          // we don't have a user record with this ID, make a new record!
          new User({ googleId: profile.id })
            .save()
            .then(user => done(null, user));
        }
      });
    }
  )
);
```

If a user already exists in the MongoDB Database, run the 'done()' function (first argument will be an error, if there is any, and the second argument is the user to add to the req.user object from here on in.

5. navigate to 'http://localhost:5000/api/current_user'. This will show you

```javascript
app.get('/api/current_user', (req, res) => {
    res.send(req.user);
});
```


<h5>Instructions for creating keys for the <a href="config/dev.js">dev.js config file</a></h5>

1. Create a Google App: Get the Client_ID and Client_Secret

2. Create a MongoDB Database - you can create on locally, or via https://mlab.com/ 

Mlab will provide you with a MONGO_URI. It will look something like this:

mongodb://<dbuser>:<dbpassword>@ds123372.mlab.com:23372/amp-it-up

replace <dbuser> and <dbpassword> with credentials for the DB User you create via Mlab.

3. Create a random COOKIE_KEY. This will be used for NPM's cookieSession library (allows you store user data directly in the cookie you pass back and forth to the browser).

Here's you use this var within the main server file (index.js):

```
app.use(
  cookieSession({
    maxAge: 30 * 24 * 60 * 60 * 1000,
    keys: [keys.cookieKey]
  })
);
```

<h5>Once Authentication Works</h5>

NPM's Passport library automatically adds several methods to every incoming request from a logged-in user:

```javascript
req.user // ------> Get Data of the Logged In User
req.logout() // ------> Deletes Session
req.session // ------> accesses cookie from browser - session info
```

<h5>Remember:</h5>

a) cookieKey var needs to be hidden (don't push it to git), otherwise, there's a danger of someone decrypting cookies you issue your users.

b) When you deploy to heroku, Heroku automatically sets the NODE_ENV variable equal to 'production'. This explains what we're doing in the <a href="config/keys.js">config/keys.js file</a>


<h5>Viewing the flow charts</h5>

You'll notice in the <a href="diagrams/"> folder you have a lot of flowcharts. You can access them at: https://www.draw.io/