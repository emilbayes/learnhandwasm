# 03. Multiplication

Now it is time to write you own WAT module. We will start very simple, as to
get you used to the syntax. A popular topic for conversation is the weather,
however when talking to my American friends I can never understand what they
mean because they use this scale called Fahrenheit. So let's make conversation
easier by building a converter!

The conversion formula is `c = (f - 32) * 5 / 9` for Fahrenheit to Celsius and
`f = (c * 9 / 5) + 32` the other way around. I have put in a couple of extra
parenthesis to make the task a little less bug-free. The type signature for your
functions should use `f64`, and you may use the `f64.div: [f64, f64] -> [f64]`,
`f64.sub: [f64, f64] -> [f64]`, `f64.const: [num|hex] -> [f64]` and
`f64.mul: [f64, f64] -> [f64]` instructions. Remember `(get_local $name)` from
the previous module.

The typing notation here is not standard, but looks like the one you may
encounter in the spec, and eg. the one for `f64.const` means you can give it a
number or a hex number like `0xff`. Sometimes you may encounter instructions
that take one type and return another like `f64.eq: [f64, f64] -> [i32]`.

One surprising detail is that you cannot just write numbers inline in WASM, like
`(i32.add 2 2)` but have to explicitly define these numbers as constants:
 `(i32.add (i32.const 2) (i32.const 2))`


The exercise is to write the functions `f64.f2c` for Fahrenheit to Celsius and
`f64.c2f` for the other way around. You can use the following test cases:

```js
var wasm = require('./temperature')

var mod = wasm()

console.log(mod.exports['f64.f2c'](0) === -17.77777777777778)
console.log(mod.exports['f64.c2f'](-17.77777777777778) === 0)
console.log(mod.exports['f64.f2c'](100) === 37.77777777777778)
console.log(mod.exports['f64.c2f'](37.77777777777778) === 100)

console.log(mod.exports['f64.f2c'](-459.66999999999996) === -273.15)
console.log(mod.exports['f64.c2f'](-273.15) === -459.66999999999996)
```

<details>
  <summary><strong>Solution</strong></summary>

```webassembly
;; temperature.wat
(module $temperature
  (func $f64.f2c (export "f64.f2c")
    (param $f f64)
    (result f64)
    (f64.mul
      (f64.div (f64.const 5)
               (f64.const 9))
      (f64.sub (get_local $f)
               (f64.const 32))))
  (func $f64.c2f (export "f64.c2f")
    (param $c f64)
    (result f64)
    (; It's not supposed to be that easy :) ;)))
```

```sh
wat2js temperature.wat -o=temperature.js -- --debug-names
```
</details>

When ready head on to next exercise:

[**Exercise 05**](../05)
