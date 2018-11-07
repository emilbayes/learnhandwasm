# 07. A detour on debugging

So far we have trusted our ability to write correct code, but soon we will go
into more looping, which is notorious for causing bugs, so I want to do a detour
on how I do debugging in WASM.

## Disassembly

So far we have only covered the S-expression format, but there is actually
another format, which is the one that your WAT is compiled into, called the
linear format. It carries with it a short description of how WASM works.
In practise WASM is based around a stack machine. This means our initial
example of `$i32.add` will compile to be:

S-Expression:

```webassembly
(module $arithmetic
  (func $i32.add (export "i32.add")
    (param $a i32)
    (param $b i32)
    (result i32)

    (i32.add
      (get_local $a)
      (get_local $b))))
```

Linear format:

```webassembly
(module $arithmetic
  (func $i32.add (export "i32.add")
    (param $a i32)
    (param $b i32)
    (result i32)

    get_local $a
    get_local $b
    i32.add))
```

The idea is that an instruction can pop it's arguments off a virtual stack,
and push a value back onto the stack for the next instruction. In the above
example `get_local $a` pushes an `i32` onto the stack, `get_local $b` pushes an
`i32` onto the stack, making it two `i32`s on the stack, and then `i32.add` pops
two `i32`s and pushes an `i32` onto the stack. This matches the `result` of the
function. This is why having two `add`s in Exercise 02 was compiler error.

Current Firefox and Chrome will disassemble WASM into the linear format in their
DevTools if you go to the "Sources" tab, but in slightly different ways.
They will also allow you to inspect the stack of values when hitting a
breakpoint and step though the instructions, though this can be flaky in Chrome,
in my experience.

Chrome 69 Debugger:

```webassembly
func $i32.add (param i32 i32) (result i32)
  get_local 0
  get_local 1
  i32.add
end
```

Firefox 63 Debugger:

```webassembly
(module
  (type $type0 (func (param i32 i32) (result i32)))
  (export "i32.add" (func $i32.add))
  (func $i32.add (;0;) (param $var0 i32) (param $var1 i32) (result i32)
    get_local $var0
    get_local $var1
    i32.add
  )
)
```

## Logging

However sometimes you just want to log a value, whether as a separate
instruction or inline in a call to see a value at a particular point in time.

For this I always define `$t.log: [t] -> []` and `$t.log_tee: [t] -> [t]`.
The former will log any one value eg.

```webassembly
(call $i32.log (get_local $counter))
```

or log inline by inspecting the value and putting it back on the stack:

```webassembly
(set_local $counter
           (call $i32.log_tee
                 (i32.add (i32.const 8)
                          (call $i32.log_tee
                                (get_local $counter))
```

Notice how this will first log the initial value of `$counter` before
multiplication and the product after multiplication, before updating `$counter`,
by simply injecting `(call $i64.log expr)`. This will always work since WASM
functions (as of current) can only push a single value on the stack at a time.

### How it's done

To achieve logging we need to call out to Javascript. For this we need to import
a function.

This can be done like this:

```webassembly
(module $debug_helpers
  (func $i32.log (import "debug" "log") (param i32))
  (func $i32.log_tee (import "debug" "log_tee") (param i32) (result i32)))
```

Notice how you need two strings when importing and you need to define the
function signature when importing.

From Javascript with `wat2js` you can then do:

```js
var wasm = require('./debug-module')

var mod = wasm({
  import: {
    debug: {
      log (...args) {
        console.stack(...args)
      },
      log_tee (arg) {
        console.stack(arg)
        return arg
      }
    }
  }
})
```

This will make the two functions available for WASM to call.

The full example for all WASM types is given here:

```webassembly
(module $debug_helpers
  (func $i32.log (import "debug" "log") (param i32))
  (func $i32.log_tee (import "debug" "log_tee") (param i32) (result i32))
  ;; No i64 interop with JS yet - but maybe coming with WebAssembly BigInt
  ;; So we can instead fake this by splitting the i64 into two i32 limbs,
  ;; however these are WASM functions using i32x2.log:
  (func $i32x2.log (import "debug" "log") (param i32) (param i32))
  (func $f32.log (import "debug" "log") (param f32))
  (func $f32.log_tee (import "debug" "log_tee") (param f32) (result f32))
  (func $f64.log (import "debug" "log") (param f64))
  (func $f64.log_tee (import "debug" "log_tee") (param f64) (result f64))

  ;; i64 logging by splitting into two i32 limbs
  (func $i64.log
    (param $0 i64)
    (call $i32x2.log
      ;; Upper limb
      (i32.wrap/i64
        (i64.shl (get_local $0)
                 (i64.const 32)))
      ;; Lower limb
      (i32.wrap/i64 (get_local $0))))

  (func $i64.log_tee
    (param $0 i64)
    (result i64)
    (call $i64.log (get_local $0))
    (return (get_local $0))))
```

```js
var wasmModule = require('./debug_helpers')

wasmModule({
  import: {
    debug: {
      log (...args) {
        console.stack(...args)
      },
      log_tee (arg) {
        console.stack(arg)
        return arg
      }
    }
  }
})
```

To use this, paste the WASM part into `debug_helpers.wasm`, then copy the JS
above into each JS file.

## Asserting

Another good technique to catch bugs is being very defensive when coding, which
can be done by making assertions of certain facts in a program. In Node.js we
have the `assert` module providing a suite of utilities for asserting various
facts. In WASM there's a `unreachable` instructions which will cause an
unconditional trap, meaning the execution of the program stops, just like throw
in Javascript. The trap will be propagated to the Javascript engine and
converted to an `RuntimeError`.

For assertions I use the following snippet:

```
(func $assert (param i32) (if (i32.eqz (get_local 0)) (unreachable)))
```

Now I can make assertions in my code:

```webassembly
(module $arithmetic
  (func $assert (param i32) (if (i32.eqz (get_local 0)) (unreachable)))

  (func $i32.add (export "i32.add")
    (param $a i32)
    (param $b i32)
    (result i32)

    ;; assert that $a is less than 10
    (call $assert (i32.le_u (get_local $a) (i32.const 10)))

    (i32.add
      (get_local $a)
      (get_local $b))))
```

When the trap is converted to a Javascript error, the error message will have
`assert` in the stack trace, letting me know that the error was caused by an
assertions.

Important to note, just as in Javascript, is that any mutations done before an
error will persist after, but this exacerbated by the fact that everything is
written to global memory.

[**Exercise 08**](../08)
