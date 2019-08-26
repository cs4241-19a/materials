# Quiz:

# Discuss points from reading

A couple of imoprtant things to point out:

1. Loose equality : don't use it! If you know when to use it and want to use it deliberately (I do this rarely), leave a comment explaining your decision so other programmers (including your future self) don't think it's a mistake

2. `typeof null => 'object` : ARGGGGGGGH! Oh, JavaScript, oh JavaScript.

3. There two ways to access object variables: *dot notation* and *bracket notation*

4. `typeof [] => 'object'` : ARGGGGGGH! Oh, JavaScript. However, you can use the `Array.isArray()` function.

5. == vs. ===, truthy and falsy.

6. Functional (lexcial) scoping: where a variable is declared in the program text determines where it's accessible from.
  - dynamic scoping: scoping depends upon the program state or execution context.
  
7. Hoisting; An annoying JavaScript concept. Best to declare variables at the top of functions when possibble.

8. "use strict"
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode

```
// creates variable on global namespace
!function() { test = 'blah' }()

// throws error
!function() { 
  'use strict'
  test2 = 'blah' 
}()
```

9. Simple prototype demo
```
const ourprototype = { a: 1 }
const ourobject = Object.create( ourprototype )
console.log( ourobject.a ) // => 1 
```

10. Simple closure demo / using the debugger in Chrome
```
const outer = function() {
  const thisWillBeEnclosed = 42
  
  const inner = function() {
    console.log( thisWillBeEnclosed )
    debugger
  }
  
  return inner
}

const inner = outer()
inner()
```

11. Show some expressions in https://astexplorer.net

# Adding to our server from last class

Three goals:
1. We want to send out updated files when they've been changed as opposed to only loading them when the server first starts.
2. We want to handle all file types, not just the index.html file.
3. Add correct MIME types to packets (Multi-purpose Internet Mail Extensions)

## Handling file updates

- Show old server file by browsing the GitHub commit history
- Change to reading file on demand

```
const http = require('http'),
      fs   = require('fs'),
      port = 3000

const server = http.createServer( function( request,response ) {
  switch( request.url ) {
    case '/':
      sendFile( response, 'index.html' )
      break
    case '/index.html':
      sendFile( response, 'index.html' )
      break
    default:
      response.end( '404 Error: File Not Found' )
  }
})

server.listen( process.env.PORT || port )

const sendFile = function( response, filename ) {
   fs.readFile( filename, function( err, content ) {
     file = content
     response.end( content, 'utf-8' )
   })
}
```

## Handle all file types
Creating a MIME-type function:

```js
const mimeForExt = function( ext ) {
  let mime = null
  switch( ext ) {
    case 'png': case 'gif':
      mime = 'image/' + ext
      break
    case 'jpeg': case 'jpg':
      mime = 'image/jpeg'
      break
    case 'htm': case 'html':
      mime = 'text/html'
      break
    case 'css':
      mime = 'text/css'
      break
    case 'js':
      mime = 'text/javascript'
      break
    default:
      mime = 'text/plain'
      break
  }   

  return mime
}
```

- Instead, use 'mime' library via npm.
  - show npm website and how to install library in Glitch console
    - explain --save flag
    - point out changes to package.json after installation
    
- Show how to strip leading forward slash from request.url with String.slice()
- Create generic function that serves any file with appropriate MIME type
  - Show MIME types in Network tab of Chrome console
- Remember to handle the domain-name only ('/') exeception!
- Provide proper error code

## Final Server
```
const http = require( 'http' ),
      fs   = require( 'fs' ),
      // IMPORTANT: you must run `npm install` in the directory for this assignment
      // to install the mime library used in the following line of code
      mime = require( 'mime' ),
      port = 3000

const server = http.createServer( function( request,response ) {
  // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/slice
  const filename = request.url.slice( 1 ) // remove leading forward slash (**not a backslash**)

  console.log( filename )
  
  switch( filename ) {
    case '':
      if( request.url === '/' ) sendFile( response, 'index.html' )
      break
    default:
      sendFile( response, filename )
      break
  }
})

server.listen( process.env.PORT || port )
``
const sendFile = function( response, filename ) {
   // mime types: https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types
   const type = mime.getType( filename ) 

   fs.readFile( filename, function( err, content ) {

     // if the error = null, then we've loaded the file successfully
     if( err === null ) {

       // status code: https://httpstatuses.com
       response.writeHeader( 200, { 'Content-Type': type })
       response.end( content )

     }else{

       // file not found, error code 404
       response.writeHeader( 404 )
       response.end( '404 Error: File Not Found' )

     }
   })
}
```

# HTML / CSS

1. There are four must have elements to create a valid / well-formed HTML page:
- !doctype
- <html lang='en'>
- <head>
- <meta charset='utf-8'>
- <body>
  
2. Semantic HTML
  - https://developer.mozilla.org/en-US/docs/Glossary/Semantics
  - previously there were primarily many less meaningful elements, like <span>, <div>, and <font>
  - show non-semantic charlie-roberts.com
    - discuss viewing individual elements and associated CSS
    
3. Applying CSS
  - If we're only using semantic HTML, how do we control the presentation of the content?
  - where can CSS go?
    - `style` attribute in individual tags (worst)
    - `<style>` tag in head (better)
    - external stylesheets (best)
  - selectors: tag, id, and class
  - most specific rule gets precedence!
  

