# Unit Testing in JS

In the examples below we'll write unit tests using both vanilla Node.js and the [Mocha testing framework](mochajs.org). We'll be testing the following piece of code, which performs math on two numbers

```js
module.exports = {
  add( n1, n2 ) { return n1 + n2 },
  sub( n1, n2 ) { return n1 - n2 },
  mul( n1, n2 ) { return n1 * n2 },
  div( n1, n2 ) { return n1 / n2 }
}
```

## Vanilla unit tests

Really, you don't *need* anything extra in JavaScript to run unit tests except for [Node's builtin `assert` command](https://nodejs.org/api/assert.html). Here's a snippet to test that file:

```js
const assert = require( 'assert' )
const math   = require( 'math' )

assert.equal( math.add( n1, n2 ), n1 + n2, 'it cannot add' )
assert.equal( math.sub( n1, n2 ), n1 - n2, 'it cannot sub' )
assert.equal( math.mul( n1, n2 ), n1 * n2, 'it cannot mul' )
assert.equal( math.div( n1, n2 ), n1 / n2, 'it cannot div' )

// if it gets here, it passes
console.log( 'all unit tests pass!)
```

`assert.equal` checks to see if the first argument is equal to the second; if not, it prints the error message you pass as your third argument to the scene. `assert` also has functions like `.deepEqual` that performs a deep check to see if two objects are equivalent (they contain the same data, but potentially stored at different memory locations.)

## Unit testing with Mocha

`mocha` is a framework that produces a more readable output for your unit tests, and automates the testing process for you. You can install it globally using `npm i mocha`. It basically is just a fancy wrapper around `assert`, with extra syntax for describing and logging your unit tests. When tests are run with `mocha` you get some extra methods added to node without having to explictly include them; in the example below these are `describe` and `it`.

Here's a test suite using Mocha:

```js
const assert = require( 'assert' )
const math   = require( '../math.js' )

define( 'Math', () => {
  it( 'should add two numbers without error' ()=> {
    assert.equal( math.add( n1, n2 ), n1 + n2 )
  })
  it( 'should subtract two numbers without error' ()=> {
    assert.equal( math.sub( n1, n2 ), n1 - n2 )
  })
  it( 'should add multiply two numbers without error' ()=> {
    assert.equal( math.mul( n1, n2 ), n1 * n2 )
  })
  it( 'should div two numbers without error' ()=> {
    assert.equal( math.div( n1, n2 ), n1 / n2 )
  })
})
```

By default, `mocha` will look for a directory named `test` and expect all your unit tests to be stored there. You can have as many test files as needed in this drectory, and it makes sense to create seperate files for each feature set you're etsting. Assuming you've setup your project wiht a `testt` directtory, you can then run `mocha` from the top level of your project and you should see output similar to thte following:

```
  Math
    ✓ should add two numbers without error
    ✓ should subtract two numbers without error
    ✓ should add multiply two numbers without error
    ✓ should div two numbers without error


  4 passing (3ms)
```

You can also enable different styles of reporting (I like the [Nyan style](https://www.youtube.com/watch?v=QH2-TGUlwu4)):

`mocha --reporter nyan`

Once you've defined some unit tests, you can easily setup `npm` to run them for you by adding the following to the `package.json` file for your project:

```js
"scripts": {
  "test": "mocha"
}
```

Now you can just run `npm test` to test your project with `mocha`.

# React, take 2

To get started with React, follow [the instructions provided by Facebook](https://github.com/facebook/create-react-app/) to initialize a new app using the `create-react-app` package. 

First, we require a basic file that loads our `App` component into a particular DOM element (index.js). Below is the file given by `create-react-app`; you shouldn't need to modify this file typically, although we'll play with it a bit for this demo.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(<App />, document.getElementById('root'));

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

If you run `npm start` from within your app directory, you should see a webpage suggesting that you modify the `App.js` file. OK, `create-react-app` gives us a simple App.js file to start with. Let's edit to start from a relatively blank slate:

```js
import React from 'react';
import './App.css';

function App() {
  return (
    <div className="App">
      TESTING
    </div>
  );
}

export default App;
```

If we look at our `index.js` file we can see that it's importing our `App` component and then rendering it to a `<div>` named root. 
  
## Using properties (props)
We can easily set it up so that we can pass values from our instantation of `<App />`:
  
`ReactDOM.render(<App text="DEMO TEXT"/>, document.getElementById('root'));`

OK, so now we're passing a property named `text` to our `App` component. By default, React will a dictionary of any properties defined on a component to the constructor for that component. So all we have to do to display the associated text is change our `App` constructor slightly, and then reference these properties within our rendered HTML.

```js
import React from 'react';
import './App.css';

function App( props ) {
  return (
    <div className="App">
      { props.text }
    </div>
  );
}

export default App;
```

We use the `{}` brackets to indicate that JavaScript should be evaluted and injected into the rendered DOM element.

Note that if you want to use classes, you can also define your App like this:

```js
import React from 'react';
import './App.css';

class App extends React.Component {
  render() {
    const div = <div className="App">
      { this.props.text }
    </div>
    
    return div
  }
}

export default App;
```

We'll do this in the future when we start dealing with `state`.

## More components

Let's make another simple component and call it `Header`. Create a file named `Header.js` in the `src` directory of your React app.

```js
import React from 'react'
function Header( props ) {
  return (
    <h1>{ props.text }</h1>
  )
}
export default Header
```

Now let's insert that into our `App`:

```js
import React from 'react'
import Header from './Header'
import './App.css';

function App( props ) {
  return (
    <div className="App">
      { props.text }
      <Header text='My Header 1' ></Header>
      <Header text='My Header 2' ></Header>
      <Header text='My Header 3' ></Header>
    </div>
  )
}

export default App;
```

OK! We're now passing data between multiple components that we've defined. Excellent.

## State

These property values are not allowed to change over time. In order have dynamic values, we must create `state`. Even state is basically immutable; in order to "change" it we have to reassign an entirely new object to the state of a component... we cannot modify properties of a components state directly. In order to use state, we need to define constructor and initialize it.

```js
import React from 'react'
import './App.css'

class App extends React.Component {
  constructor( props ) {
    super( props )
    this.state = { anumber:0 }
    this.tick()
  }
  render() {
    const div = <div className="App">
      { this.props.text }<br />  
      { this.state.anumber }
    </div>
    
    return div
  }
  tick() {
    setInterval( ()=> this.setState({ anumber:this.state.anumber + 1 }), 1000 )
  }
}

export default App
```

In the. above example, we define a `tick` method that calls React's `.setState` method. Note that we pass `setState` an entirely new object representing the current state of our component. If you look at your page now, you should see that the number increments around once a second. The HTML is automatically update to reflect the current state.
