# 05. Writing Arrays

So now we can convert between the two scales, but only one temperature at a
time. The message spreads far and wide that we have made this new high
performance solution for converting between Fahrenheit and Celsius, that we are
now asked if we can work on large scale datasets. However, our function only
take a single data point at a time. We know from Javascript that we would just
take an array of data points and convert them in one big batch, so let's see
how this is done in WASM. This will introduce us to linear memory and blocks.

## Linear memory

So in Javascript all memory is automatically managed for you, but in WebAssembly
we have to manage this by ourselves. We are even more low-level here than in C,
as there is no memory allocator, but simply one big piece of continuous memory.
You can think of this like a big `Buffer` from Node or an
`ArrayBuffer`/`Uint8Array` like in Browsers.

From a module we have to explicitly define that we want to use memory,
beyond the simple parameters we had available in our previous functions. We
also need to explicitly say that we will make this memory available outside the
module, so eg. Javascript or other WASM modules can access it.
This is done like this:

```webassembly
(module $memory
  (memory (export "memory") 1)
  (; functions here ;))
```

The signature is `(memory $name? (export|import)? initialPages maxPages?)`.
You can currently only have one "memory" in a WASM module, and it is allocated
in "pages", which are 64kB. As the signature implies, you can also import
external memory! Fancy! However we will not look at this for now. You can also
define a maximum number of pages that you want the memory to be able to
allocate. There is an implied maximum of 4GB of RAM as we will see in just a
moment.

Also note it is important in what order you define various top-level constructs
such as functions and memory. Simply define memory as the first thing and you
should be good :)

### Storing data

To store data we need to learn a bit about how memory works. Let's create a
function that can store Numbers for us. The memory addressed at a byte level,
which is why you can think of it as a bit byte array. Addressing is simply a
`i32` index into the memory, often also called a pointer. For our function
below we want to pretend the memory is a big array of Numbers, and as `f64`
hints, this is 64-bits wide. `64 bits = 8 bytes` which means to go from an
index to a pointer, we need to multiply by 8.

Here is a table showing what the memory would look like with the f64 `1` at
address 0 and the f64 `10e-100` at address 8:

| Address | +0         | +1         | +2         | +3         | +4         | +5         | +6         | +7         |
|:--------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|:-----------|
| 0       | `00000000` | `00000000` | `00000000` | `00000000` | `00000000` | `00000000` | `11110000` | `00111111` |
| 8       | `00111110` | `11000011` | `11011000` | `01001110` | `01111101` | `01111111` | `01100001` | `00101011` |
| 16      | `00000000` | `00000000` | `00000000` | `00000000` | `00000000` | `00000000` | `00000000` | ...        |


As you can see the representations are quite funky. This is due to the floating
point standard used by Javascript called IEEE 754.

We could write a simple helper that lets us treat our memory as a large array
of `f64`s.

```webassembly
;; memory.wat
(module $memory
  (memory (export "memory") 1)

  (func $f64.set (export "f64.set")
    (param $index i32)
    (param $value f64)
    (result f64)

    (local $ptr i32)

    (set_local $ptr (i32.mul (i32.const 8)
                             (get_local $index)))

    (f64.store (get_local $ptr)
               (get_local $value))))
```

There are a couple of new instructions above. I introduced a `local`, which is
like a variable, to hold our pointer. As mentioned, we need to multiply our
index by 8 to convert to an `f64` address. Locals must be defined at the
beginning of a function in WASM, just like `params`. We then store the value
into memory at the pointer.

Compiling the above:

```sh
wat2js memory.wat -o=memory.js -- --debug-names
```

And running in Javascript:

```js
var wasm = require('./memory')

var mod = wasm()

mod.exports['f64.set'](0, 1) // set 1 at address 0
mod.exports['f64.set'](1, 10e-100) // set 10e-100 at address 8

console.log(mod.exports.memory) // You should be able to see the table above
```

Check the above and see what the resulting memory looks like.

### Exercise

Write the function `$f64.get: [i32] -> [f64]` that reads a Number from a index
by multiplying up to the right address. You can use the instruction of the name
`f64.load: [i32] -> [f64]` which is almost equivalent to `f64.store`.

Test case:

```js
var wasm = require('./memory')

var mod = wasm()

mod.exports['f64.set'](0, 1)
mod.exports['f64.set'](1, 10e-100)

console.log(mod.exports['f64.get'](0) === 1)
console.log(mod.exports['f64.get'](1) === 10e-100)
```

[**Exercise 06**](../06)
