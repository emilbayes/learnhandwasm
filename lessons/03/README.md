# 03. WASM <3 Javascript

So far we have only looked at WASM code at rest and compiled from WAT, but we
are yet to see how to interact with WASM from Javascript, which is what we are
all so excited about!

At this point you should have `wat2js` installed globally. Otherwise go ahead
and fetch it from npm. We can now compile and wrap our `arithmetic.wat` in a
CommonJS module:

```sh
wat2js arithmetic.wat -o=arithmetic.js
```

If you want to pass options to `wat2wasm`, it can be done as this:

```sh
wat2js arithmetic.wat -o=arithmetic.js -- --debug-names
```

`--` is a convention to split args. This can also be used with eg. npm scripts
to pass args to the underlying command, eg `npm start -- --port=8080`. Anyway!

You will now have you `arithmetic.js` module, which is a light-weight wrapper
around our WASM module, making it a bit easier to use. There are other tools
doing this like `wasm-bindgen`, transforms for Webpack and Browserify etc.
However in this workshop we will only use this tool and later look at how to
instantiate modules manually.

To use our module you can require it from Javascript:

```js
// require the compiled module as you would any other js module
var wasm = require('./arithmetic')

// instantiate the wasm
var mod = wasm()

// and call an exported wasm function from js
console.log(mod.exports['i32.add'](10, 10))
```

A couple of things to note:

* The wrapper module exports a single function. This functions takes some
  arguments that we will look at later, eg. an object of imports and whether the
  module should load sync or async
* When initialised, the wrapper returns an object with a couple of convenience
  properties:
    * `mod.buffer` is the binary WASM module as a `Uint8Array`
    * `mod.memory` will be the exported linear memory as a `Uint8Array` if
      present
    * `mod.exports` contains all the exported "things" from the module under
      their given name. There is no namespacing as `.` may have suggested when
      reading the source. There are other elements than functions that can be
      exported from a WASM module
    * Exported functions are directly callable from Javascript

Try executing the following commands in context of the script above:

```js
console.log(4294967295 + 3735928559)
console.log(mod.exports['i32.add'](4294967295, 3735928559))
console.log(4294967295 + 3735928559 >>> 0)

console.log(7.09 + 5.381)
console.log(mod.exports['i32.add'](7.09, 5.381))
console.log(7.09 + 5.381 >>> 0)
```

* What results did you get?
* What happened? Why?
* What is `>>>`?

[**Exercise 04**](../04)
