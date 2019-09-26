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


## Server

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