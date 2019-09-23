# Going pro: browserify, babel, gulp, and linting

## JavaScript Modules
There are a variety of systems for modularizing JS code, and a huge number of libraries have been created over the users for this. The simplest method is to check for the existence of a global object and place our code into this object. If it doesn’t exist, create it first. For example:

```js
if( typeof window.app === 'undefined' ) window.app = {}

window.app.mymodule = {
  foo:1,
  bar() { console.log( this.foo ) }
}
```

Using this strategy, we can safely import any number of files via script tags and not have to worry about the order that they’re loaded in, assuming only one of them makes use of `window.onload` to start our app running.

However, this can lead to some spaghetti code. What if one particular module needs to know about the existence of another module so that it can call functions from it? We need some sort of dependency system for this. `browserify` is one such tool for the client.

### Before we get to browserify, what about importing / exporting in node.js?
Node.js conveniently includes a `require` function to include files and a `module.exports` object for exporting objects / functions / values. This method of exporting / requiring modules is known as `CommonJS`. We've used require a lot in our servers, but you might not have used `module.exports` yet:

```js
/************ FILE 1: module.js, a module to export **********/

module.exports = function() {
  console.log( 'a function' )
}

/************ FILE 2: importing our module ************/
const demomodule = require( './module.js' )

demomodule() // logs 'a function'
```

Any module can `require` any other module, and node.js will resolve circular dependencies whenever possible.

### Browserify is a module system with the same syntax as node.js, but for the browser
* In order to avoid having to resolve asynchronous dependency loading, browserify flattens all JS files used in a project into a single JS file that can then be imported into a webpage with a single `<script>` tag.
* Browserify also provides browser-compatible versions of many commonly used libraries in node.js, such as its event, buffer, and crypto libraries.
* Let’s try using browserify to create a three.js application.

### Demo w/ Three.js and Browserify
1. First, install browserfiy. `npm install browserify --g`. Adding the `--g` flag will make it a global install that we can call directly from our shell. If you get an error about permissions, try the steps outlined in [this article](https://medium.com/@ExplosionPills/dont-use-sudo-with-npm-still-66e609f5f92)
2. Open some type of bash shell. Create a new directory `mkdir mydir` and cd into it.
4. Install three.js via npm. `npm install three`
5. OK, let’s make our main html file. It will be almost identical to previous three.js demo, except this time we’ll be including single JS file that browserify will create for us (named `bundle.js`), while last time we explicitly included three.js in a script tag.

```html
<!doctype html>
<html lang='en'>
  <head>
    <style> body { margin:0 } </style>
    <script src='./bundle.js'></script>
  </head>
  <body></body>
</html>
```

6. Now we need to create our main JavaScript file; let’s name it `main.js`. In a later step, browserify will compile this into a file named bundle.js that’s included by our HTML file. Again, we’ll use our code from last week, with one key modification:

```js
// import our three.js reference
const THREE = require( 'three' )

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

7. If you compare the script above to last week’s three.js code, the only difference is the line at the top of the code, where we import our primary THREE object to use; last week we had a global reference to the main THREE object instead. As we’ll see in a moment, our THREE variable Is *not* placed in the global namespace when we require it with browserify this way.
8. Let’s run browserify! We need to identify our main JavaScript file, and then specify what we want our output file to be called: `browserify main.js -o bundle.js`
9. Load up your HTML file. Uh-oh, you might have a bug.  Browserify can introduce bugs if you don’t properly identify your character set using a `<meta>` tag. Add the following to the `<head>` of your HTML file if you're having problems:

`<meta charset='utf-8'>`

10. Everything should be running at this point if you load your HTML. Note that if you open up your developer console and type THREE it is undefined… it's not available in the global namespace.
11. Let’s add another library, one that makes it a bit easier to add post-processing effects to THREE.js. We’ll use this to add a glitching effect to our scene. The postprocessing library doesn’t actually add anything new to three.js, it just makes it easier to access and use some of the post processing objects.
12. In three.js we can add post processing effects using the `EffectComposer` object, which enables us to stack multiple effects over a scene as needed. Each effect is a fragment shader that is run inside of a `RenderPass` object. In order to use the EffectComposer we need to create two RenderPass objects: the first will render our scene (the spinning cube with the light) and the second one will apply our glitch effect.
13. First, install the post-processing library: `npm install postprocessing`
14. Now we'll change our `main.js` to the following:

```js
const THREE = require( 'three' )
const PP    = require( 'postprocessing' )

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
    
    this.createEffects()
    this.clock = new THREE.Clock()
    this.render = this.render.bind( this )
    this.render()
  },
  
  createEffects() {
    this.composer = new PP.EffectComposer( this.renderer )
    this.renderPass = new PP.RenderPass( this.scene, this.camera )
    this.composer.addPass( this.renderPass )

    this.glitchPass = new PP.EffectPass( this.camera, new PP.GlitchEffect() )
    this.glitchPass.renderToScreen = true
    this.composer.addPass( this.glitchPass )
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
    this.composer.render( this.clock.getDelta() )
    window.requestAnimationFrame( this.render )
  }
}

window.onload = ()=> app.init()
```

15. OK, you should now have a running glitch effect on your spinning cube. One annoying aspect about browserify that you probably noticed is the need to re-run the browserify command anytime you make changes to your JavaScript. We can fix this by using `watching`, which will automatically re-run browserify for you anytime you change a relevant JS file. 
	1. Install watchify with `npm install watchify -g`
	2. Run watchify: `watchify main.js -o bundle.js -v`

Now whenever the file is changed and browserify is called you’ll see an update message appear in your terminal.

### ES6 import / export
ES6 added `import` and `export` keywords, however, it's only been in the last year or so that browsers have added support, and [in node.js it's still an experimental feature](https://thecodebarbarian.com/nodejs-12-imports). However, it's definitely the future of modules in JavaScript, so we'll try using it here and explain how to make it work in all browsers with `browserify` and a plugin for transpiling, `babelify`.

#### The basics
1. Let's make a basic module that exports a single number and name it `module.js`:
```js
export default 42
```

The `default` keyword tells us that `42` is the standard export for this file. You can only have one `default` export per module, however, we'll see soon that named exports enable us to export multiple values.

2. Let's create an `app.js` file that imports this value:
```js
import anumber from `module.js`

console.log( 'anumber is:', anumber )
```
3. OK, so we can include `app.js` in a web page with two extra steps:
   - We need to run a server with CORS enabled (e.g. http-server . -p 30000), can't load via `file://`
   - We need to add the `type="module"` attribute to our `<script>` tag.

```html
<!doctype html>
<html lang='en'>
<head>
  <script src='app.js' type='module'></script>
</head>

<body></body>

</html>
```

4. If you open the page and developer's console, you should see the number `42` printed.

#### Slightly more complex
In this next example, we'll export a file named `utilities.js`, that will provide us with a couple of different functions to use. We'll export these using `named` exports, instead of `default` exports like we did in our previous example.

```js
// utilities.js
const log = ( obj, prop ) => console.log( `${prop}: ${obj[ prop ]}`)

const square = num => num * num

export { log, square }
```

Now we can import and use those functions in our `app.js` file:

```js
// app.js
import { log, square } from './utilities.js' 

const app = { myvalue: square(42) }

log( app, 'myvalue' )
```

We can import `app.js` using the same HTML / server we used in the previous example. In the next example we'll look at how to make sure this script can run in all browsers, using `browserify` and `babel`, a transpiler for JavaScript.

## Working universally (almost)
As of today, only [87% of browsers support ES6 modules](https://caniuse.com/#feat=es6-module). In order to support all browsers, we need to use `browserify` on our `utilities.js` script. However, `browserify`—[and node in general](https://thecodebarbarian.com/nodejs-12-imports)—doesn't support `import` and `export`. In order to get our script into browserify, first we have to *transpile* our JavaScript to use CommonJS. The transpiling tool we'll use is called `Babel`, and is used for everything from transpiling TypeScript to JavaScript and compiling the `.jsx` extension used by the React framework.

One nice side effect of using Babel here is that we can also use it to convert our JavaScript to ES5, ensuring that it will run in even more browsers. This can basically be done "for free", assuming you don't use certain advanced JS features (like Proxies) that can't be emulated.

1. First we need to install `babelify`: `npm i babelify`. You should see a warning when you do this, something like `babelify@10.0.0 requires a peer of @babel/core@^7.0.0 but none is installed. You must install peer dependencies yourself.`
2. `babelify` is what is known as a browserify *transform*, that fits into the browserify pipeline and modifies values passing through it in some way. In this case, `babelify` modifies data going through browserify by wrapping the `babel` module, which is why we received that warning. Let's go ahead and install babel: `npm i @babel/core`.
3. Last but not least we need a babel *preset* that will tell us how we want to transpile our code. There are presets for TypeScript, React, and many other systems, but here we want a preset that will convert our code to ES5. This preset is named `@babel/preset-env`, install it now: `npm @babel/preset-env`. `preset-env` will let you [define specific browsers that you want to target](https://babeljs.io/docs/en/env/) (Netscape 3, anyone?) but the  defaults are fine for our purposes.
4. OK, we're ready to run `browserify`. We need to specify that we want to run it with a transform (`-t`) and then specify both the transform and the preset we want to use: `browserify js/index.js -o bundle.js -t [ babelify  --presets [@babel/preset-env] ]`. *IMPORTANT*: the spacing in the arrays (this is the syntax for [node.js subargs](https://www.npmjs.com/package/subarg)) is *NOT OPTIONAL*. Make sure you enter it correctly!
5. We can now import our `bundle.js` file using a regular `<script>` tag, no `type='module` attribute required. If you look at the code, you might be able to see that our ES6 specific JS has also been replaced.

## Gulp: a build system for JavaScript
There are [a variety of different build tools for web apps](https://2018.stateofjs.com/other-tools/build_tools). The one I prefer is Gulp, however, Parcel is also really interesting and seems to, for the most part, require almost zero configuration. The reason I like gulp is that you can explicitly define every step of your pipeline, as opposed to having another process handle it using default mechanisms you may not be familiar with.

Gulp typically operates on node.js `streams`, which divide data into chunks and pass them through various stream operations. These operations can read, write, and transform the stream in various ways. Additionally, gulp extends regular node.js streams using metadata called Vinyl. A Vinyl stream is designed to deal with processing files, and therefore contains extra metadata about aspects of the file, like its name. 

Unfortunately, not every operation that we might use in a build system can run on streams; sometimes we need the whole file to perform certain types of data processing. Minifying JavaScript is one example of this; it's impossible to minify with fragments of JavaScript code, as you need to be able to see the entire file to understand what variable names can be safely shortened. Because of this, we have to `buffer()` streams (convert the stream into a file) before we pass the data to certain functions.

### Basic gulp/babel/browserify setup
We can use browserify/babelify from within gulp. Let's start with that basic setup:

```js
// import libraries installed via NPM
// note: to use babelify you must have installed
// @babel/core and @babel/preset-env
const gulp   = require( 'gulp' ),
      source = require( 'vinyl-source-stream' ),
      browserify = require( 'browserify' ),
      babelify   = require( 'babelify' )

const build = function() {
  browserify({ entries:'./js/index.js' })
    .transform( "babelify", { presets: ["@babel/preset-env"] })
    // create Node.js stream using browserify output
    .bundle()
    // add Vinyl meta-data to Node.js stream, in this case
    // the name of the file we're streaming
    .pipe( source('bundle.js') )
    // output file to dist folder
    .pipe( gulp.dest('./dist') )
}

// 'default' task runs whenever we call 'gulp' on the 
// command line with no arguments
gulp.task( 'default', gulp.series( build ) )
```
Note that `gulp.task` defines what happens when we call `gulp`. We can define a bunch of different tasks and then call each of them from the command line using `gulp taskname`, where task name is the first argument we pass to `gulp.task()`. The value `default` is used to enable calling `gulp` with no arguments on the command line.

### Fancier setup with minification and notifications
```js
// import libraries installed via NPM
const gulp   = require( 'gulp' ),
      rename = require( 'gulp-rename' ),
      uglify = require( 'gulp-uglify' ),
      notify = require( 'gulp-notify' ),
      source = require( 'vinyl-source-stream' ),
      buffer = require( 'vinyl-buffer' ),
      browserify = require( 'browserify' ),
      babelify   = require( 'babelify' )

const main = function () {
  // browserify our data
  const out = browserify({ entries:'./js/index.js' })
    // apply transform to convert js to es5
    .transform( "babelify", { presets: ["@babel/preset-env"] })
    // create Node.js stream using browserify output
    .bundle()
    // add Vinyl meta-data to Node.js stream, in this case
    // the name of the file we're streaming
    .pipe( source('bundle.js') )
    // output file to dist folder
    .pipe( gulp.dest('./dist') )
    // collect chunks into a single buffer for
    // buffer-based processing that operates on files
    .pipe( buffer() )
    // minimize our buffer
    .pipe( uglify() )
    // rename our file for output
    .pipe( rename ('bundle.min.js') )
    // send minimized output to dist folder
    .pipe( gulp.dest('./dist') )
    // create notification that build is complete
    .pipe( notify({
      message: 'build complete.',
      onLast:  true
    }) )
    
  return out
}

gulp.task('build', gulp.series( main ) )

// watch our js folder and trigger build task
// whenever a .js file is changed
gulp.task( 'watch', gulp.series( 
  function() {
    gulp.watch( './js/**.js', function() {
      main()
    })
  })
)

// define default task whenever 'gulp' is run
gulp.task( 'default', gulp.series('build') )
```

In the example above, we use `buffer` to buffer our stream into a single piece of data that can then be minified using the `uglify()` plugin. We then rename the file and output it to our `./dist` folder; everytime we run our script we'll generate both a minified and unminified version of the file.

## Linting
Linting lets us spot potential errors / inefficiencies in our code and also ensure that it has a unified format across each project. Common packages for this include [standard.js](https://standardjs.com) and [eslint](https://eslint.org/docs/user-guide/command-line-interface). Standard.js requires less configuration and is available as a plugin for most text editors / IDEs, however, it is not customizable. To use it:

1. install it: `npm i standard`
2. `cd` into a directory with javascript files.
3. run it with `npx`(node package executed): `npx standard`
4. See the list of errors and fix what you want. Or call `npx standard --fix` to try to automatically fix most of them.
5. We can also [plug it into gulp](https://www.npmjs.com/package/gulp-standard).

[Read about the drama surrounding funding standard via adds in the terminal](https://feross.org/funding-experiment-recap/).

There is also [semistandard (standard.js with semicolons)](https://github.com/standard/semistandard). YMMV.