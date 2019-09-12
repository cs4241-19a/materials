# No quiz!
- We'll be doing some work with Promises in class today
- Assignment #3 is due on Monday(!)

# Passport.js and user authentication
[Passport.js](http://passportjs.org) provides a standardized way to authenticate users using a variety of different techniques. For this assignment, we are focusing on using username / password combinations that are stored in a database (for most of you, using lowdb).

There are lots of different ways to get hung up integrating Passport the first time you use it. We'll try and talk through these.

## First, understand the basic flow of information we'll be using
```
Client: Send username / password to Passport > 

< Server: Look up username / password combination
  - if successfully matched, redirect or send information to client
    - AND also set cookie
  - if failed, redirect back to login page with reason for failure

Client: Request some other piece of user information
  - Once authenticated, all requests sent to the client will contain a session cookie!
  
< Server: Using the session cookie received from the client, match each request to an internal table of authenticated users
  - If user is found, attach user information to `request` object going through express middleware
    - and deliver requested information
```    

There are many additional steps that we could insert here to improve the user experience, but this will suffice for this assignment.

## Second, let's import our modules and get a basic Express server going
The modules we'll use (at a minimum) are:

- `express`: handle routing and middleware for processing data
- `express-session`: helps attach cookies to responses via middleware
- `passport`: the main authentication library
- `passport-local`: passport features hundredes of authentication `Strategies`, where each strategy implements a different authentication method. The `Local` strategy is named because it only uses data residing in the server; for example, it doesn't make a call to an external API like Google or Facebook for authentication.
- `body-parser`: express middleware for parsing JSON data. *IMPORTANT GOTCHA THAT YOU SHOULD AVOID*: You have to ensure that any data sent from the client includes the `Content-Type: application/json` header, otherwise this middleware *WILL NOT WORK* and the entire server will break without providing useful error messages.

Given that, here's our basic server setup for delivering static routes ("static" routes are routes that do not have any dynamic dependencies, such as resolving password authentication or performing other user data processing.)

```js
const express   = require( 'express' ),
      app       = express(),
      session   = require( 'express-session' ),
      passport  = require( 'passport' ),
      Local     = require( 'passport-local' ).Strategy,
      bodyParser= require( 'body-parser' )

app.use( express.static('./') )
app.use( bodyParser.json() )

// server logic will go here

app.listen( process.env.PORT || 3000 )
```

## The simplest HTML/JS possible for this, for testing
Hopefully you got a sense of how forms work in the last assignment; here we'll avoid GUIs for authentication (you'll need to have one for the assignment though!) to keep things simple, and immediately make a `fetch` call in a `<script>` element to the server. For purposes of this example, this page will be named `index.html` and placed in the server, however, it normally makes sense to store these files in a separate directory (the default Express app in Glitch names this `/public`).
  
```html
<!doctype html>
<html lang='en'>
  <head>
    <meta charset='utf-8'>
    <style>body { background:#000; }</style>
  </head>

  <body></body>

  <script>
    fetch( '/login', {
      method:'POST',
      body:JSON.stringify({ username:'charlie', password:'charliee' }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( res => res.json() )
    .then( console.log )
  </script>
</html>
```

A couple of things to note:

- We have included our `Content-Type` header. *Remember to do this or you will cry yourself to sleep at night in frustration*. This ensures that our `body-parser` Express middleware will parse the JSON we send from the client.
- We use `Promises` to process our data asynchronously. `res => res.json()` will be called *when the headers for our JSON data have arrived*, the resulting promise return by `.then` will be resolved once the body of the JSON has been received in its entirety. The line `.then( console.log )` is (almost) functionally identical to `.then( response => console.log( response ) )`; in both cases we're passing a function that will be called when the promise is resolved... but `.then( console.log )` is a nice shortcut.

## OK, so we send a username and a password to the server, what next?
Let's add a route to handle `POST` requests to `/login`, and get our basic Passport authentication going. The code below should be appended to our initial server code, where the comment `// server logic goes here` is placed.

```js
// a simple table to store non-persistent data. for assignment #3
// your data must be persistent between sessions using a database (lowdb)
const users = [
  { username:'charlie', password:'charliee' },
  { username:'bill',    password:'billl' }  
]

// all authentication requests in passwords assume that your client
// is submitting a field named "username" and field named "password".
// these are both passed as arugments to the authentication strategy.
const myLocalStrategy = function( username, password, done ) {
  // find the first item in our users array where the username
  // matches what was sent by the client. nicer to read/write than a for loop!
  const user = users.find( __user => __user.username === username )
  
  // if user is undefined, then there was no match for the submitted username
  if( user === undefined ) {
    /* arguments to done():
     - an error object (usually returned from database requests )
     - authentication status
     - a message / other data to send to client
    */
    return done( null, false, { message:'user not found' })
  }else if( user.password === password ) {
    // we found the user and the password matches!
    // go ahead and send the userdata... this will appear as request.user
    // in all express middleware functions.
    return done( null, { username, password })
  }else{
    // we found the user but the password didn't match...
    return done( null, false, { message: 'incorrect password' })
  }
}

passport.use( new Local( myLocalStrategy ) )
passport.initialize()

app.post( 
  '/login',
  passport.authenticate( 'local' ),
  function( req, res ) {
    console.log( 'user:', req.user )
    res.json({ status:true })
  }
)
```

If we load our `index.html` page, open the developer's tool and look at the Network tab, we should see some information going back and forth in addition to the the `{ status:true }` JSON that is printed to console.log.

## Eating our cookies
OK! We have authentication! Now how do we use that authentication process to ensure that all requests sent from the client arrive at the server with credentials showing that authentication takes place? The answer lies in the nefarious cookie. If we set a cookie using our server after authentication happens, we'll be able to use that cookie to identify our user. Creating this type of system, where cookies are used to provide credentials, is typically called a "session". in server terminology.

To use cookies with passport we need a few things;

- A function to `serialize` each user (store it in the session) when they are authenticated
- A function to `deserialize` each user (retrieve it from the session) when a request is received from the client. If a user is found, it will be added to `request.user` in all express middleware functions.

These functions are usually extremely small, we want to use a piece of information that will uniquely identify each user.

```js
passport.serializeUser( ( user, done ) => done( null, user.username ) )

// "name" below refers to whatever piece of info is serialized in seralizeUser,
// in this example we're using the username
passport.deserializeUser( ( username, done ) => {
  const user = users.find( u => u.username === username )
  console.log( 'deserializing:', name )
  
  if( user !== undefined ) {
    done( null, user )
  }else{
    done( null, false, { message:'user not found; session not restored' })
  }
})
```

Great! Now we have simple functions that will save our user to a session and lookup users from sessions when rqeuests arrive. Our last step is to provide the middleware needed to process the cookies. We'll also add a new route, `/test` to check and make sure our cookies are working.

```js
app.use( session({ secret:'cats cats cats', resave:false, saveUninitialized:false }) )
app.use( passport.initialize() )
app.use( passport.session() )

app.post('/test', function( req, res ) {
  console.log( 'authenticate with cookie?', req.user )
  res.json({ status:'success' })
})
```

## Final testing
OK, so now we want to see if our cookies work. After our `index.html` loads, let's just make a call to `fetch()` from inside the developer console.

```js
fetch( '/test', {
  method:'POST',
  credentials: 'include'
})
.then( console.log )
.catch( err => console.error ) 
```

The important part to note is the `credentials:'include` in the options for fetch. This tells our fetch request to include the session cookie (which is initially set after successful authentication via a call to `/login`) when it makes the request. If we don't include these credentials, the server won't receive the cookie and the session will be inactive.

Also, remember, if you add JSON to your request, *YOU MUST ADD THE Content-Type HEADER TO YOUR FETCH OPTIONS*.

# Three.js
- [http://threejs.org](http://threejs.org)
- One of two popular 3D libraries for the web, with the other being Babylon.js
- Simple HTML to include three.js from a content delivery network:

```html
<!doctype html>
<html lang='en'>
  <head>
    <meta charset='utf-8'>
    <style> body { margin: 0 } </style>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/three.js/108/three.min.js'></script>
    <script src='./main.js'></script>
  </head>
  <body></body>
</html>
```

OK, it does take a bit of setup to get three.js up and running. Here's a script that creates a camera, lights, and a spinning geometry:

```js
const app = {
  init() {
    this.scene = new THREE.Scene()

    this.camera = new THREE.PerspectiveCamera()
    this.camera.position.z = 50 

    this.renderer = new THREE.WebGLRenderer()
    this.renderer.setSize( window.innerWidth, window.innerHeight )

    document.body.appendChild( this.renderer.domElement )
    
    this.createLights()
    this.knot = this.createKnot()

    // ...the rare and elusive hard binding appears! but why?
    this.render = this.render.bind( this )
    this.render()
  },

  createLights() {
    const pointLight = new THREE.PointLight( 0xffffff )
    pointLight.position.z = 100

    this.scene.add( pointLight )
  },

  createKnot() {
    const knotgeo = new THREE.TorusKnotGeometry( 10, .1, 128, 16, 5, 21 )
    const mat     = new THREE.MeshPhongMaterial({ color:0xff0000, shininess:2000 }) 
    const knot    = new THREE.Mesh( knotgeo, mat )

    this.scene.add( knot )
    return knot
  },

  render() {
    this.knot.rotation.x += .025
    this.renderer.render( this.scene, this.camera )
    window.requestAnimationFrame( this.render )
  }
}

window.onload = ()=> app.init()
```


