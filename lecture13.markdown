# FINAL PROJECTS
- In Riley Commons

## CS420X - Graphical Simulation of Physical Systems - C Term

- GPU-based computing
  - Show CPU-based Reaction Diffusion equation (Turing) at 200x200...
  - ... vs GPU-based RD equation at 512x512
  
- Analog video synthesis, feedback, fluid simulations

# What we hopefully learned

- Basics of writing a server
- Basics of accessing / storing data
- Basics of Using HTML / CSS
- Basics of JavaScript
- JS Classes are evil (jk)
- Modern JS toolchains are a pain
  - We have to browserify / watchify
  - We have to gulp
  - We have to mocha
  - We have to babel
  - etc. etc.

# Trends in JS that we haven't talked about

## Compile targets
  - Briefly discussed using Babel for ES6 > ES5 and React conversion
  - Other toolchains:
    - Any language to WebAssembly
      - DOOM3 in the browser: http://wasm.continuation-labs.com/d3demo/

### Typescript
  - Strong typing added to JavaScript
  - Pros:
    - Can catch errors at *write / compile time*, instead of at runtime
    - Can provide more confidence that code is "correct"
  - Cons:
    - I think it looks kinda ugly, but that's me
    - You have to buy into the whole toolchain thing
  - `npx create-react-app my-typescript-app --typescript`
  - [More info on integrating with React](https://www.pluralsight.com/guides/typescript-react-getting-started)
  
#### Porting a simple React app to use TypeScript
```
import React from 'react';
import './App.css';

interface TodoProps { 
  name:string, 
  completed:boolean, 
  onclick: (name:string, checked:boolean) => void 
}

class Todo extends React.Component< TodoProps > {
  render() {
    return <li>{this.props.name} : 
      <input type="checkbox" defaultChecked={this.props.completed} onChange={ e => this.change(e) }/>
    </li>
  }
  change(e: {target:any}) {
    this.props.onclick( this.props.name, e.target.checked )
  }
}

let headers: any = { method:'get', 'no-cors':true }
class App extends React.Component<{}, { todos:Array<Todo> }> {
  constructor( props: any ) {
    super( props )
    this.state = { todos:[] }
    this.load()
  }

  load() {
    fetch( '/read', headers )
      .then( response => response.json() )
      .then( json => {
         this.setState({ todos:json }) 
      })
  }

  toggle( name:string, completed:boolean ) {
    fetch( '/change', {
      method:'POST',
      body: JSON.stringify({ name, completed }),
      headers: { 'Content-Type': 'application/json' }
    })
  }
 
  add() {
    const value:string = document.querySelector('input')!.value

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

  render() {
    return (
      <div className="App">
      <input type='text' /><button onClick={ e => this.add()}>add</button>
        <ul>
          { 
            this.state.todos.map( (todo:any,i) => {
              return <Todo key={i} name={todo.name} completed={todo.completed} onclick={ this.toggle } />;
            }) 
          }
       </ul> 
      </div>
    )
  }
}

export default App;
```

### Other languagess
- Elm (Haskell-ish)
  - https://elm-lang.org
- ClojureScript (Clojure / LISP)
- Reason (OCaml)
  
## Multi-threaded JavaScript
  - As Moore's law fails, multithreaded apps are becoming more of a necessity
    - especially true for Android devices

  - Web Workers
    - Post messages between threads
    - Can run WebAssembly (although not required)
    - https://github.com/mdn/simple-web-worker
    
## Compiling CSS
  - [Usage statistics](https://2019.stateofcss.com/technologies/pre-post-processors/)
  - [Preprocessor comparison](https://raygun.com/blog/css-preprocessors-examples/)
  - Yet more tooling, it will never end...
  - Also, JS -> CSS

## Full-stack development
  - Modular components FTW?
  - Salaries on average are about 10% higher than front-end devs (although less than dedicated back-end developer)
  - Especially desirable by "lean" (aka cheap) startups
  - "The United States Bureau of Labor Statistics estimates full-stack development employment to increase from 135,000 to over 853,000 by the year 2024."
    - Source: https://hackernoon.com/full-stack-developer-a-popular-in-demand-tech-job-in-2019-ehxs3z6y    