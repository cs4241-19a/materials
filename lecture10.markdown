# A4 due tomorrow

# Noëlle + d3

# Components
- Improve modularity by dividing application UI elements into components
- In most cases, combine HTML + JS (and sometimes CSS) into single file definitions
  - Angular maintains HTML/JS/CSS separation
  
https://2018.stateofjs.com/front-end-frameworks/overview/

- For the following examples, we'll use the server at the end of these notes. The server manages
a simple todo list using express + lowdb.

## Svelte
http://svelte.dev
- Uses code generation to create a small, optimized application
- Not made by Facebook(!)

### init a new svelte app
https://github.com/sveltejs/template
```
npx degit sveltejs/template svelte-app
cd svelte-app
```

### edit our App.svelte file
A file to load/display data from our server:
```html
<script>
  const getTodos = function() {
    const p = fetch( '/read', {
      method:'GET' 
    })
    .then( response => response.json() )
    .then( json => {
      console.log(json)
      return json 
    })
 
    return p
  }
  
  let promise = getTodos()
</script>
  
{#await promise then todos}
  <ul>

  {#each todos as todo}
    <li>{todo.name} : <input type='checkbox' todo={todo.name} checked={todo.completed}></li>
  {/each}

  </ul>
{/await}  
```

The part inside the `<script>` tag should look fairly normal. Outside of it is pretty weird looking though. The first line `{#await promise then todos}` basically is saying. Additionally, it's also saying "Everytime the promise resolves (re)create this list." Then, we start an unordered list `<ul>`. Next we're saying "for each todo in our todos variable, create a list item." We can see that we can insert JavaScript expressions inside of the `{}` characters, similar to how in ES6 we can put them expressions between `${ }` inside of template strings.
  
### adding new todos (reactive programming)
Here's where the magic happens:

```html
<script>
  const getTodos = function() {
    const p = fetch( '/read', {
      method:'GET' 
    })
    .then( response => response.json() )
    .then( json => {
      console.log(json)
      return json 
    })
 
    return p
  }

  const addTodo = function( e ) {
    const todo = document.querySelector('input').value
    promise = fetch( '/add', {
      method:'POST',
      body: JSON.stringify({ name:todo, completed:false }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( response => response.json() )
  }
  </script>


<input type='text' />
<button on:click={addTodo}>add todo</button>

{#await promise then todos}
  <ul>

  {#each todos as todo}
    <li>{todo.name} : <input type='checkbox' todo={todo.name} checked={todo.completed} on:click={toggle}></li>
  {/each}

  </ul>
{/await}
```

In the above code we're adding the `addTodo` function and a button that triggers it. The magic is that, simply by redefining our `promise`. our list is recreated everytime we a new todo. The UI is *reactive* to changes in the underlying data. OK, let's finish by adding a checkbox to toggle whether each todo item has been completed.


```html
<script>
  const getTodos = function() {
    const p = fetch( '/read', {
      method:'GET' 
    })
    .then( response => response.json() )
    .then( json => {
      console.log(json)
      return json 
    })
 
    return p
  }

  const addTodo = function( e ) {
    const todo = document.querySelector('input').value
    promise = fetch( '/add', {
      method:'POST',
      body: JSON.stringify({ name:todo, completed:false }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( response => response.json() )
  }

  const toggle = function( e ) {
    fetch( '/change', {
      method:'POST',
      body: JSON.stringify({ name:e.target.getAttribute('todo'), completed:e.target.checked }),
      headers: { 'Content-Type': 'application/json' }
    })
  }

  let promise = getTodos()
</script>`.


<input type='text' />
<button on:click={addTodo}>add todo</button>

{#await promise then todos}
  <ul>

  {#each todos as todo}
    <li>{todo.name} : <input type='checkbox' todo={todo.name} checked={todo.completed} on:click={toggle}></li>
  {/each}

  </ul>
{/await}
```
Here we're simply submitting a POST to request whenever each checkbox is checked/unchecked to update the data on the server. The UI is automatically changed via normal HTML behavior, however, the UI will also reflect changes to the data when the page is refreshed.

## React
For comparison, here is roughly the same file created in React. You can create a basic React project (using Babel and Parcel) by following [these instructions](https://createapp.dev/parcel). Alternatively, you can follow [the instructions provided by Facebook](https://github.com/facebook/create-react-app/).

First, we require a basic file that loads our `App` component into a particular DOM element (index.js). Below is the file given by `create-react-app`; you shouldn't need to modify this file.

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

Next we create our `App.js` file. This file will be converted into valid JS by Babel, but React enables us to author
our components by freely mixing HTML and JS. 


```js
import React from 'react';
import './App.css';

// we could place this Todo component in a separate file, but it's
// small enough to alternatively just include it in our App.js file.

class Todo extends React.Component {
  // our .render() method creates a block of HTML using the .jsx format
  render() {
    return <li>{this.props.name} : 
      <input type="checkbox" defaultChecked={this.props.completed} name2={this.props.name} onChange={ e => this.change(e) }/>
    </li>
  }
  // call this method when the checkbox for this component is clicked
  change(e) {
    this.props.onclick( this.props.name, e.target.checked )
  }
}

// main component
class App extends React.Component {
  constructor( props ) {
    super( props )
    // initialize our state
    this.state = { todos:[] }
    this.load()
  }

  // load in our data from the server
  load() {
    fetch( '/read', { method:'get', 'no-cors':true })
      .then( response => response.json() )
      .then( json => {
         this.setState({ todos:json }) 
      })
  }

  // when an Todo is toggled, send data to server
  toggle( name, completed ) {
    fetch( '/change', {
      method:'POST',
      body: JSON.stringify({ name, completed }),
      headers: { 'Content-Type': 'application/json' }
    })
  }
 
  // add a new todo list
  add( evt ) {
    const value = document.querySelector('input').value

    fetch( '/add', { 
      method:'POST',
      body: JSON.stringify({ name:value, completed:false }),
      headers: { 'Content-Type': 'application/json' }
    })
    .then( response => response.json() )
    .then( json => {
       this.setState({ todos:json }) 
    })
  }

  // render component HTML using JSX 
  render() {
    return (
      <div className="App">
      <input type='text' /><button onClick={ e => this.add( e )}>add</button>
        <ul>
          { this.state.todos.map( (todo,i) => <Todo key={i} name={todo.name} completed={todo.completed} onclick={ this.toggle } /> ) }
       </ul> 
      </div>
    )
  }
}

export default App;
```

## Server
The server below applies for both the Svelte and React projects above.

```js
const express  = require( 'express' ),
      app      = express(),
      low      = require( 'lowdb' ),
      FileSync = require( 'lowdb/adapters/FileSync' )
 
const adapter = new FileSync('db.json')
const db = low( adapter )

db.defaults({ 
  todos:[
    { name:'buy groceries', completed:false }
  ]
}).write()

app.use( express.static( 'public' ) )
app.use( express.json() )

app.get( '/read', function( req, res ) {
  res.json(
    db.get('todos').value()
  )
})

app.post( '/add', function( req,res ) {
  const todos = db.get('todos').push( req.body ).write()
  res.json( todos )
})

app.post( '/change', function( req,res ) {
  const user = db
    .get('todos')
    .find({ name:req.body.name })
    .set( 'completed', req.body.completed )
    .write()
  
  res.sendStatus( 200 )
})

app.listen( 8080 )
```
