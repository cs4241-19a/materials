# Quiz

# HTML / CSS

## Slides
  - Ted Nelson, Xanadu, and Hypertext
    - Precursor: Vannevar Bush and the Memex
  
  - Markup Languages
    - TeX, 1978 by Donald Knuth
    - GML, developed at IBM
      - -> SGML
        - HTML & XML
  
  - HTTP
    - OSI reference model
    - HTTP sits on top of TCP and IP
      - Big difference between TCP and UDP?
      
  - Browser History
    - 1990: Tim Berners-Lee creates first web server and web browser, WorldWideWeb
    - 1993: Marc Andreessen's Mosaic (soon Netscape)
    - 1995: Internet Explorer
    - 2006: Firefox
    - 2007: Safari
    - 2008: Chrome
  
  - CSS
    - First proposed in 1994 by Håkon Wium Lie
    - 2000: Internet Explorer 5 for the Mac first browser to implement over 99% of the CSS 1 spec
      - discrepancies between browsers and quirks in IE led to new browser development in the mid-2000s

  - JavaScript Introduction
    - First developed in 1995 by Brandon Eich
    - Standardized by the European Computer Manufacturers Associationn
    - 2011: ES5
    - 2015: ES6 (or ES2015)
  
1. There are four must have elements to create a valid / well-formed HTML page:
- `doctype`
- `<html lang='en'>`
- `<head>`
- `<meta charset='utf-8'>`
- `<body>`
  
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
  - View (and edit!) CSS in the developers console.

# Assignment 2 Starter Pack

## Server

- Server is mostly the same as the improved server from the last assignment
- But now we will handle both GET and POST requests

```js
const server = http.createServer( function( request,response ) {
  if( request.method === 'GET' ) {
    handleGet( request, response )    
  }else if( request.method === 'POST' ){
    handlePost( request, response ) 
  }
})

const handlePost = function( request, response ) {
  let dataString = ''

  request.on( 'data', function( data ) {
      dataString += data 
  })

  request.on( 'end', function() {
    console.log( JSON.parse( dataString ) )

    // ... do something with the data here!!!

    response.writeHead( 200, "OK", {'Content-Type': 'text/plain' })
    response.end()
  })
}
```

## Client
  
```html
<!doctype html>
<html lang="en">
  <head>
    <title>CS4241 Assignment 2</title>
    <meta charset="utf-8">
  </head>
  <body>
    <form action="">
      <input type='text' id='yourname' value="your name here">
      <button>submit</button>
    </form>
  </body>
  <script>

  const submit = function( e ) {
    // prevent default form action from being carried out
    e.preventDefault()

    const input = document.querySelector( '#yourname' ),
          json = { yourname: input.value },
          body = JSON.stringify( json )

    fetch( '/submit', {
      method:'POST',
      body 
    })
    .then( function( response ) {
      // do something with the reponse 
      console.log( response )
    })

    return false
  }

  window.onload = function() {
    const button = document.querySelector( 'button' )
    button.onclick = submit
  }

  </script>
</html>
```
