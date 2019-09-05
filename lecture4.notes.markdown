# Day 4

## CSS Positioning
You don't need to know all the details / properties, but do know:

- That flexbox exists, and makes it easy to define responsive layouts.
- That grid exists, and makes it easy to create layouts that are automatically organized into a grid without having to worry about rows / columns in your HTML.
- That absolute and fixed positioning exist, letting you position your content in ways that "ignore" the currently used CSS flow model.
- In class on Tuesday we'll look at discuss some CSS frameworks and templates that handle most layout/positioning aspects for you.

## Updates on the server for assignment 2
- Where to place the server logic
- What is REST? How could we structure or server to use a REST API?
- Data does not need to be persistent inbetween server sesssions!
- Another way to pass data: Cookies
  - automatically sent to server if cookie is found with *every request*
  - can also be used to remember signin information
  - https://plainjs.com/javascript/utilities/set-cookie-get-cookie-and-delete-cookie-5/
  - https://www.npmjs.com/package/cookies
  - multipage web apps vs. single page web apps

## Some tips on accessibility
- Use HTML 5 semantic tags (https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
  - how does a screen reader find the navigation of your website? mark it up!
- alt attribute for all images, embedded videos etc.
  - show viewing in lynx as "extreme" example
- To JavaScript or not to JavaScript...
- Contrast! Use contrast checker on the developer console (demo)
  - Also remember color blindness in this. It is often not ideal to use color as a sole distinguishing feature (e.g. for links)
- Design acheivment for assignment 2: consider accessibility, use alt tags for all images, check for contrast on relevant elements, use semantic tags instead of `<div>` whennever possible.

## The end of scope and closures (and a bit of DOM manipulation)
- Hopefully the last chapter of the book on scope cleared up why it's important, and how scope and closure is in JavaScript.
```js
const btn = document.createElement('button')
const a = 'test'
btn.onclick = function() { console.log( 'The value of a is ' + a + ', captured in a closure.' ) }
document.body.appendChild( btn )
```

Basically, *every* time you define an event handler, you are creating a closure that's capturing the surrounding lexical scope. What are examples in our server from the current assignment?

https://github.com/cs4241-19a/a2-shortstack/blob/master/server.improved.js

The classic for loop problem using var:

```js
// clear webpage
document.body.innerHTML = ''

for( var i = 0; i <5; i++ ) {
  const btn = document.createElement('button')
  btn.innerHTML = i
  btn.onclick = function() { console.log( 'The value of i = ' + i  ) }
  document.body.appendChild( btn )
}
```

We can fix this by using `let` instead of `var`, which provides block scoping:

```js
// clear webpage
document.body.innerHTML = ''

for( let i = 0; i <5; i++ ) {
  let j = i
  const btn = document.createElement('button')
  btn.innerHTML = j
  btn.onclick = function() { console.log( 'The value of i = ' + j  ) }
  document.body.appendChild( btn )
}
```

... and then `let` makes things even easier when used with `for` loops:

```js
// clear webpage
document.body.innerHTML = ''

for( let i = 0; i <5; i++ ) {
  const btn = document.createElement('button')
  btn.innerHTML = i
  btn.onclick = function() { console.log( 'The value of i = ' + i  ) }
  document.body.appendChild( btn )
}
```

What about "modules" and providing private scope?

```js
const MyConstructor = function() {
  const privateVariable1 = 'this is top secret'
  
  const obj = {
    doSomething() { console.log( privateVariable1 ) }
  }
  
  return obj
}

const myObj = MyConstructor()
console.log( myObj.privateVariable1 ) // => undefined
console.log( myObj.doSomething() )    // => 'this is top secret'
```

- As described in the readings for today, ES6 has a nice syntax for importing JS modules. However: https://caniuse.com/#feat=es6-module-dynamic-import


## this and binding

- The rules are annoying, but once you learn them, lots of JS suddenly makes sense (similar to rules for scope).
- Default Binding:
```js
function defaultBinding() {
  console.log( this )
}

defaultBinding()
```

- Implicit Binding:
```js
const obj = {
  implicit() { console.log( this ) }
}

obj.implicit()
```

- Explicit Binding:
```js
const obj = {
  myfunc() { console.log( this ) }
}

obj.myfunc.call( { test:1 } )
```

- Hard Binding:
```js
const obj = {
  myfunc() { console.log( this ) }
}

const obj2 = {
  testing:1
}
obj2.myfunc = obj1.myfunc.bind( obj2 )
```

- Lexical Binding:
```js
const obj = {
  myfunc() { 
    setTimeout( ()=> console.log( this ), 1000 )
  }
}

obj.myfunc()
```

- `new` Binding:
```js
const aconstructor = function() {
  console.log( this )
}

new aconstructor()
```

## JavaScript Objects (chapter 3 of the reading)

- All good information, but the most important point is probably that there is no built-in way to easily duplicate JS objects
- `Object.assign()` will get you a shallow object, and that is good enough for many situations.


