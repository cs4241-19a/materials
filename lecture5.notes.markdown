# Extra Credit Quiz

# Gibber properties
- Example bizzaro use of object setters and getters, like those discussed in chapter 3 of this & object.prototypes.

# Object prototypes, the new keyword, and Function.prototype madness

## What does the new keyword do?
Given the following code:
```js
const Robot = function( name ) {
  this.name = name
}
Robot.prototype.speak = function() {
  console.log( `My name is ${this.name}.` )
}
const bobBot = new Robot( 'Bob' )
bobBot.speak() // => My name is Bob.
```
... what does the use of `new` accomplish when creating our new `Robot`?

1. Automatically creates a new object and binds it to the value of the `this` keyword inside our Robot function
2. Sets the prototype of our new object to whatever is found in the `Robot.prototype` property
3. The value of `this` is automatically returned by our `Robot` function

*INCREDIBLY IMPORTANT:* all three of the above rules only apply if the `new` keyword is used in front of our invocation of the `Robot` function!

What happens if we run: `const maryBot = Robot( 'Mary' )` after the code above?

## Avoiding the magic
That's a lot of behind-the-scenes automagickal nonsense that happens. How can we avoid it and still use prototypes? `Object.create`

```js
const a = { foo:42 }
const b = Object.create( a )
console.log( a.foo ) // => foo!
```

Note that this method employs *none* of the hand-waving magic that the `new` keyword exploits. Once you how prototypes work, it is cleaner and easier to understand. 

## OK, so then what's a mixin? 
JavaScript only lets you have one prototype chain for delegation. In order to use multiple objects (you can think of this as faux multiple inheritance) you can use mixins. In the example below, we mixin behaviors for movement and logging into objects that defer to `RobotProto`.
```js
const Mover = {
  move( x,y=0,z=0 ) {
    this.x += x
    this.y += y
    this.z += z
    return this
  },
  reset() {
    this.x = this.y = this.z = 0
    return this
  }
}

const Logger = {
  log( ...args ) {
    args.forEach( arg => console.log( `${arg}: ${this[ arg ]}` ) )
    return this
  }
}

const RobotProto = {
  speak() { 
    console.log( `My name is ${this.name}` )
    return this
  },
  init( name ) { 
    this.name = name
    // creates shallow copies of all properties on first argument
    Object.assign( this, Mover, Logger )
    return this
  }
}

const bobBot = Object.create( RobotProto )
  .init( 'Bob' )
  .reset()
  .move( 1,2,3 )
  .speak()
  .log( 'x','y','z' )
```

## I like classes. What's the problem with classes.
No giant problems, really, and ES6 provides a nice syntax for them. However, using them hides that classes use prototypes behind the scenes... they don't behave like classes in traditional classical inheritance langauges (Java, C++ etc.). For example:

```
  class Cheap {
    constructor( value ) {
      this.value = value
    }
    cost() {
      console.log( `I cost ${this.value}.` )
    }
  }
  
  class Expensive extends Cheap {
    constructor( value ) {
      super( value )
    }
  }
  
  const pricey = new Expensive( 42 )
```

If we look at `pricey` in the developer console, we see that it has to navigate up the prototype chain *twice* to find its `cost` method!!! Using a mixin instead would prevent use of the prototype chain, which is important in frequent operations (think particle systems, or functions that operate over giant datasets).

# Express

- What is Express?
  - Provides convenient "middle-ware" services to easily handle dataflow and server logic.
  - Extremely popular, with lots of plugins that make common web tasks easy.
  
- What is middleware?
  - A simple function with the signature `function( request, response, next ){}`, where
    `next` is a function that is called when the middleware function has finished processing
    data.
  - Example: a simple logger of requested urls (adapted from express documentation).
  
```js
var express = require( 'express' )
var app = express()

app.use( function( req, res, next ) {
  console.log( 'url:', req.url )
  next()
})

app.get( '/', function (req, res) {
  res.send( 'Hello World!' )
})

app.listen(3000)
```

## Writing middleware for POST requests
- With this in mind, how would we write middleware to handle JSON information sent via POST request?

```
const express = require('express')
const app = express()
const dreams = []

// for example only: routes for handling the post request
app.use( function( request, response, next ) {
  let dataString = ''

  request.on( 'data', function( data ) {
    dataString += data 
  })

  request.on( 'end', function() {
    const json = JSON.parse( dataString )
    dreams.push( json )
    // add a 'json' field to our request object
    request.json = JSON.stringify( dreams )
    next()
  })
})

app.post( '/submit', function( request, response ) {
  // our request object now has a 'json' field in it from our
  // previous middleware
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( request.json ) )
})

const listener = app.listen( process.env.PORT, function() {
  console.log( 'Your app is listening on port ' + listener.address().port )
})
```

## But there's a middleware for grabbing  JSON, right?
- Yes indeedy. `body-parser` is the most popular one, but there are others.

- You can find middleware here: https://expressjs.com/en/resources/middleware.html

- *IMPORTANT:* For JSON data, the body-parser middleware will only take action if the data
  sent to the server is passed with a 'Content-Type' header of 'application/json'.
  For example:
```js
  fetch( '/submit', {
    method:  'POST',
    headers: { 'Content-Type': 'application/json' },
    body:    JSON.stringify( data )
  })
```

- Below below is an entire example server, less than 20 LOC not counting comments.

```js
const express    = require('express'),
      app        = express(),
      bodyparser = require( 'body-parser' ),
      dreams     = []

// automatically deliver all files in the public folder
// with the correct headers / MIME type.
app.use( express.static( 'public' ) )

// get json when appropriate
app.use( bodyparser.json() )

// even with our static file handler, we still
// need to explicitly handle the domain name alone...
app.get('/', function(request, response) {
  response.sendFile( __dirname + '/views/index.html' )
})

app.post( '/submit', function( request, response ) {
  dreams.push( request.body.newdream )
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( dreams ) )
})

app.listen( process.env.PORT )
```

- We can also tell express to only use body-parser for a particular route,
  which is usually a more efficient way to use it. To do this we simply
  pass the middleware as the second argument to our route. You can do this
  with any route/middleware combination... the `post`, `get`, `put`, `delete`
  methods of the `app` object are all overloaded.
  
```js
app.post( '/submit', bodyparser.json(), function( request, response ) {
  dreams.push( request.body.newdream )
  response.writeHead( 200, { 'Content-Type': 'application/json'})
  response.end( JSON.stringify( dreams ) )
})
```

# Basic Persistence using lowdb
- The easiest way to get data persistence in our server is to write to a file. Lowdb is one excellent library for this.
- We can choose whether we want data reads/writes to happen synchronously or asynchrnously
  - Remember that synchronous reads/writes will block the server thread! The example below is synchrnous. Use
    of asynchronous writes with `Promisees` in assignment 3 can be used as a technical achievement.

Basic setup:  
```js
const low = require('lowdb')
const FileSync = require('lowdb/adapters/FileSync')
 
const adapter = new FileSync('db.json')
const db = low( adapter )

db.defaults({ users:[] }).write()

// add a user
db.get( 'users' ).push({ name:'bob', age:42 }).write()

// filter users by age
const seniors = db.get( 'users' )
  .filter( user => user.age > 70 )
  .value()
````  