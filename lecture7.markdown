# Quiz

# Assignment #3 / extra credit extension
- Assignment 3 is due Thursday by 11:59 AM
- Extra credit is due by 11:59 PM today, 9/16
- Using unique ids (uids) in client/server transactions


OK, it does take a bit of setup to get three.js up and running. Here's a script that creates a camera, lights, and a spinning geometry.

# Creative Coding: Canvas

Most of the time when people discuss the `<canvas>` element, they're talking about the using it for 2D drawing in the browser. However, `<canvas>` is also the primary element used for 3D rendering as well. Unlike SVGs, `<canvas>` used raster rendering techniques, so the images it generates are typically not well-suited for high-resolution printing.
  
We'll run this code in our console.

```js
document.body.innerHTML = ''
canvas = document.createElement( 'canvas' )
document.body.appendChild( canvas )

// set the width and height
canvas.width = canvas.height = 500

// get our 2D drawing context, pass 'webgl' for most 3D drawing
ctx = canvas.getContext( '2d' )

// fill a rectangle: x,y,width,height
ctx.fillRect( 0,0,100,100 )

// use CSS to define colors for drawing
ctx.fillStyle = '#f00' // or rgb(255,0,0), or 'red' etc.

ctx.fillRect( 100,100,100,100 )
```

## Clearing the canvas

There are three typical ways we 'clear' the canvas:
1. by calling `ctx.clearRect( x,y,width,height )`
2. reassigning a width / height value to the canvas element
3. redrawing the background over the entire canvas

```js
// clear the canvas
ctx.clearRect( 0,0,100,100 )

// alternatively, just reassign the width/height
canvas.height = canvas.height // clears!

// redraw the background (black in this case)
ctx.fillStyle = 'black'
ctx.fillRect( 0,0,canvas.width,canvas.height )
```

## Simple animation loop
Let's setup an animation loop of a square bouncing off the borders of our canvas. We'll use `window.requestAnimationFrame` to get the highest framerate possible.

```js
document.body.innerHTML = ''
canvas = document.createElement( 'canvas' )
document.body.appendChild( canvas )

// set the width and height
canvas.width = canvas.height = 500

// get our 2D drawing context, pass 'webgl' for most 3D drawing
ctx = canvas.getContext( '2d' )

x = 0
bg = 'black'
fg = 'red'
draw = function() {
  // temporal recursion, call tthe function in the future
  window.requestAnimationFrame( draw )
  
  ctx.fillStyle = bg 
  ctx.fillRect( 0,0,canvas.width,canvas.height )
  ctx.fillStyle = fg 
  ctx.fillRect( x++,0,50,50 )
}

// must call draw to start the loop
draw()
```

How do we get a bounce? We need to add a variable that stores the direction we are traveling.

Canvas documentation on MDN: https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D

# Web Audio / FFT visualization

The browser includes an audio API that is very capable. You can either assemble graphs of C++ DSP nodes or write your own DSP routines using JavaScript. Reference: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API

```js
audioCtx = new AudioContext()
osc = audioCtx.createOscillator()
osc.connect( audioCtx.destination )

// now tell our osc oscillator to start running
// the 0 argument means start now, but we could also
// delay until some point in the future.
osc.start( 0 )

// we can change the frequency....
osc.frequency.value = 220

// we can also tell the oscillator to gradually ramp
// to a new frequency. Time is measured in seconds oscce
// the audio context was first created, to get a relative time value
// we can use the ctx.currentTime property

osc.frequency.linearRampToValueAtTime( 1760, audioCtx.currentTime + 30 )

// to stop...
// osc.stop()
```

We can change the `type` property of our oscillator to waveforms with more frequency content. Possible values include: `sine sawtooth triangle square`.

## Using gain nodes

Let's add another node to our graph to control the loudness of our oscillator. We can do this using a `GainNode`.

```js
audioCtx = new AudioContext()
osc  = audioCtx.createOscillator()
gainNode = audioCtx.createGain()

osc.connect( gainNode )
gainNode.connect( audioCtx.destination )

osc.start( 0 )

// try different values
gainNode.gain.value = .1

// osc.stop()
```

## Adding a filter
One more example, let's add a lowpass filter. This will attenuate the frequencies above a defined cutoff point in the signal.

```js
audioCtx = new AudioContext()
osc  = audioCtx.createOscillator()

// filters don't work well with sine oscillators...
osc.type = 'triangle'

gainNode = audioCtx.createGain()
gainNode.gain.value = .1

biquad   = audioCtx.createBiquadFilter()

osc.connect( biquad )
biquad.connect( gainNode )
gainNode.connect( audioCtx.destination )

osc.start( 0 )

// try different values
biquad.cutoff = 110
```

## Filter sweep
Classic lowpass filter sweep, using the same `linearRampToValueAtTime` method we used to change our oscillator frequency earlier:

```js
audioCtx = new AudioContext()
osc  = audioCtx.createOscillator()

// filters don't work well with sine oscillators...
osc.type = 'sawtooth'

gainNode = audioCtx.createGain()
gainNode.gain.value = .1

biquad   = audioCtx.createBiquadFilter()

osc.connect( biquad )
biquad.connect( gainNode )
gainNode.connect( audioCtx.destination )

osc.start( 0 )
biquad.frequency.value = 110
biquad.frequency.linearRampToValueAtTime( 1760, audioCtx.currentTime + 2 )
biquad.frequency.linearRampToValueAtTime( 110,  audioCtx.currentTime + 4 )
// change the 'quality' of the filter, which emphasizes frequencies
// near the cutoff frequency... CAREFUL. THIS CAN BLOW UP YOUR EARZ.
biquad.Q.value = 20
```

# Audio + Visuals via FFT

## Basic FFT understanding
- The sine oscillator we heard earlier is the fundamental unit of almost all audio synthesis
- Joseph Fourier proved that any wave (not just sound) can be represented by sums of sine waves with
  different frequencies and amplitudes.
- So how can we determine which frequencies are present in a wave?
  - Turns out taking the dot product of the two signals (multiplying each point in time and adding the results)
    is a good indicator, see https://jackschaedler.github.io/circles-sines-signals/dotproduct3.html
- That means we just take the dot product of every frequency we're interested in.
- The WebAudio API does this by evenly dividing the available frequencies into *bins*
  - Sampling rate = 44100 samples per second... Nyquist frequency = 22050 Hz.
  - The *fftSize* determines how many samples are used to generate each FFT result. 
  - The *frequencyBinCount* is a read-only property that is always equal to fftSize / 2.
    This gives us the nunber of different values we can use for our frequency visualization.
    - the default fftSize is 1024, for 512 bins
  - 44100 / 512 = 86 Hz per bin

## Getting an FFT with the Web Audio API

```js
// go ahead and reuse our previous code
// except lets make a much longer, wider frequency sweep
audioCtx = new AudioContext()
osc  = audioCtx.createOscillator()
osc.frequency.value = 80
osc.frequency.linearRampToValueAtTime( 880 * 4, audioCtx.currentTime + 30 )
osc.connect( audioCtx.destination )

const analyser = audioCtx.createAnalyser() // british spelling!

// set FFT size, note that 32 isn't really big
// enough to do much except use as an example
analyser.fftSize = 32
console.log( analyser.frequencyBinCount ) // > 16

// connect our sin oscillator to our analyser node
osc.connect( analyser )

// create a typed JS array to hold analysis results
const results = new Uint8Array( analyser.frequencyBinCount )

// every second, get our results and print them
const loop = setInterval( function() {
  analyser.getByteFrequencyData( results )
  console.log( results.toString() )
}, 1000 )
```

## Combine with Canvas

```js
document.body.innerHTML = ''
canvas = document.createElement( 'canvas' )
document.body.appendChild( canvas )

canvas.width = canvas.height = 512

// get our 2D drawing context, pass 'webgl' for most 3D drawing
ctx = canvas.getContext( '2d' )
audioCtx = new AudioContext()
osc  = audioCtx.createOscillator()
osc.type = 'square'
osc.frequency.value = 80
osc.frequency.linearRampToValueAtTime( 880 * 4, audioCtx.currentTime + 30 )
osc.start()
osc.connect( audioCtx.destination )

var analyser = audioCtx.createAnalyser()

analyser.fftSize = 1024 // 512 bins

// connect our sin oscillator to our analyser node
osc.connect( analyser )

// create a typed JS array to hold analysis results
var results = new Uint8Array( analyser.frequencyBinCount )

draw = function() {
  // temporal recursion, call tthe function in the future
  window.requestAnimationFrame( draw )
  
  ctx.fillStyle = 'black' 
  ctx.fillRect( 0,0,canvas.width,canvas.height )
  ctx.fillStyle = 'white' 
  
  analyser.getByteFrequencyData( results )
  
  for( let i = 0; i < analyser.frequencyBinCount; i++ ) {
    ctx.fillRect( i, 0, 1, results[i] ) // upside down
  }
}

// must call draw to start the loop
draw()
```

## Loading / visualizing audiofiles
We can create an audiofile visualizer the exact same way as above,
except instead of using an oscillator we're going to load an audiofile
using the `<audio>` element and the `MediaElementSource` node of the
Web Audio API. You can then connect that `MediaElementSource` node to
the analyser node like we did with the oscillator in our previous examples.

```js
ctx = new AudioContext()
audioElement = document.createElement( 'audio' )
document.body.appendChild( audioElement )
player = ctx.createMediaElementSource( audioElement )
player.connect( ctx.destination )
```
     
* To load a file, we set the `src` property on our audio element and
  then play the file using the play() method. However, there's a problem... CORS.
  
* ### CORS policy:
  * Can't load files from different URLs
  * Servers can selectively enable CORS.
  * You can turn it off in Chrome for particular sessions, for testing purposes
    http://www.thegeekstuff.com/2016/09/disable-same-origin-policy/
  * You can add a --cors flag with the http-server module for testing, or to use
    express middleware: https://medium.com/@alexishevia/using-cors-in-express-cac7e29b005b
    
* Once we've set up CORS, now we can load files:
  
```js
audioElement.src = 'media/somefile.mp3'
audioElement.play()
```

### All together now...

```html
<!doctype html>
<html lang='en'>
  <head></head>
  <body><button>click me</button></body>
  <script>
  const start = function() {
    document.body.innerHTML = ''
    const canvas = document.createElement( 'canvas' )
    document.body.appendChild( canvas )
    canvas.width = canvas.height = 512
    const ctx = canvas.getContext( '2d' )

    // audio init
    const audioCtx = new AudioContext()
    const audioElement = document.createElement( 'audio' )
    document.body.appendChild( audioElement )

    // audio graph setup
    const analyser = audioCtx.createAnalyser()
    analyser.fftSize = 1024 // 512 bins
    const player = audioCtx.createMediaElementSource( audioElement )
    player.connect( audioCtx.destination )
    player.connect( analyser )

    audioElement.src = 'http://127.0.0.1:20000/trumpet.wav'
    audioElement.play()

    const results = new Uint8Array( analyser.frequencyBinCount )

    draw = function() {
      // temporal recursion, call tthe function in the future
      window.requestAnimationFrame( draw )
      
      ctx.fillStyle = 'black' 
      ctx.fillRect( 0,0,canvas.width,canvas.height )
      ctx.fillStyle = 'white' 
      
      analyser.getByteFrequencyData( results )
      
      for( let i = 0; i < analyser.frequencyBinCount; i++ ) {
        ctx.fillRect( i, 0, 1, results[i] ) // upside down
      }
    }
    draw()
  }

  window.onload = ()=> document.querySelector('button').onclick = start
  </script>
</html>
```