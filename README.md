# `learnhandwasm` - Learn to Handcraft WebAssembly!

## Lesson 1

Install


```js
var wabt = require('./libwabt.js')()

var wat = ``
var features = {}

var module = wabt.parseWat('test.wast', wat, features);
module.resolveNames()
module.validate(features)
var output = module.toBinary({log: true, write_debug_names:true})

var wasm = new WebAssembly.Instance(output.buffer)

console.log(wasm.instance.exports)
```
